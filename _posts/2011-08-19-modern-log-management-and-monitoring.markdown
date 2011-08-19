---
title: Modern Log Management and Monitoring
layout: post
published: true
categories:
- Systems Administration
- Log Management
- Log Monitoring
- Monitoring
date: 2011-08-19 12:29:00
---

We've come a long way with monitoring in general - we have groups such as
\#\#monitoringsucks on Freenode who are discussing how to break down monitoring
into pieces so that we can come up with better tools to handle each component.
We have tools such as [logstash](http://logstash.net) that can structure our
logs, [graylog2](http://graylog2.org) for presenting them in real time, and
backends such as [elasticsearch](http://elasticsearch.org) for storing them in
a way that makes it easy to search for the logs again.

We're looking at problems differently than how it was seen before.  Currently,
at Seattle Biomed, we have an installation of logwatch that's on a centralized
rsyslog host that cuts down noise and sends us an email every hour, showing us
the log entries that might seem useful to us.  Unfortunately, this can
sometimes be very useless - it's easy to add in a filter that gets rid of a
bunch of noise, yet those same logs can end up being incredibly useful down the
road for catching a recurring bug.

We see this with other related tools as well: RRD's are usually used to store
metrics on events that we collect, yet it has become the de facto standard for
storing time series data, while assuming too much about how we want to store
our data (sometimes, we *don't* want to rotate our data, or we want to rotate
our data in some other way.  There's other issues too, such as assuming that
all metrics are gathered regularly - what if I wanted to graph irregular Puppet
runs?)

The tools we now have on the table look at the problem differently: collect as
much data as possible, and make them useful.  Split away functionality as much
as possible, so we have a bit more of a loosely coupled system.

What we're implementing right now in Seattle Biomed is the following:

1. logstash, for structuring different log formats into JSON, and shipping them
off to other services

2. graylog2, for presenting real time logs and divvying up the view of these
logs into different "streams"

3. elasticsearch + logstash web, for long term archival of logs and search

4. rabbitmq, for storing application log events from logstash-forwarders and
for forwarding logs between networks

To be honest, I was a bit leery about setting all of this up - I was afraid
it'd add a lot more complexity to our network.  And perhaps it did, but now
that we have something that structures any crappy logs I throw at it, something
to visualize *all* of our logs (not just syslog events), something to store
these events for long-term archival, and an AMQP broker that can also be used
for other applications, I think this little bit of added complexity has paid
off.

To show how all of this hooks up together, I hope this ASCII art drawing
tides everyone over:

<pre><code>
syslog events
-----------------> LOGSTASH -----> GRAYLOG2
                   /       \_____> ELASTICSEARCH
app events    AMQP
--------------/

</code></pre>

It'd be nice to throw everything into AMQP before having it processed by
logstash, but it looked like it required either another logstash agent (to grab
all of the syslog events and throw them into AMQP), or agents running on all
machines that watches a bunch of files.  For now, I'd rather not implement
either solution - if anyone else has other ideas, I'm all ears.  The way we
have it setup seems to work just fine though, but YMMV.

Logstash comes with support for 'grok', which is a library that allows you
to group regex's into macros, so you can write a pattern like this:

<pre><code>
%{TIMESTAMP} %{USER} %{SYSLOGPROG} %{MESSAGE}
</code></pre>

instead of writing a huge, ugly regex for everything.  You can also assign
names to a macro, which is used to structure the logs in logstash:

<pre><code>
%{TIMESTAMP:timestamp} %{USER:user} %{SYSLOGPROG:program} %{MESSAGE:message}
</code></pre>

The subsequent structure for the event will now look like this, assuming
we wrote our macros as we did above:

<pre><code>
{
  @message => <whatever was matched in MESSAGE and assigned to @message>
  @fields => {
               "timestamp" => &lt;TIMESTAMP&gt;
               "user" => &lt;USER&gt;
               "program" => &lt;SYSLOGPROG&gt;
             }
}
</code></pre>

(the patterns I've used above aren't exactly what's included in logstash, but
hopefully it gets the point across as to how powerful grok + logstash can be.)

Logstash can prevent events from going through the rest of the pipeline, but
I've decided to simply write grok patterns that structure incoming events, and
do any filtering later on in the pipeline (right now, we don't filter away
*anything* - to me, it seems better to have every message available and be able
to drill down to what entries might be relevant, instead of getting rid of a
bunch of INFO/DEBUG messages and find out later that I really needed to see
that information.)

After structuring the data with logstash, we ship the structured events off to
both elasticsearch and graylog2 using logstash's elasticsearch and GELF output
plugins.

graylog2 can match against any arbitrary key that's in the event, so if we
wanted to look at only Puppet runs, we can create a stream called "Puppet Runs"
that matches against "program=puppet-agent".  We can setup alarms and stream
subscriptions for that specific stream, thereby giving us the same sort of
functionality that we expect from logwatch and swatch.  Right now, I have three
streams for Puppet runs: "Puppet Runs" (aggregate of all Puppet Run logs),
"Puppet Runs - Changes" (if anything looks like a Puppet action, we stuff it
into this stream), and "Puppet Runs - Error" (anything that has severity level
ERROR and is a Puppet Run.)

Elasticsearch stores *every* log entry.  We capture as much as possible, so we
can look back at an entire history of logs and correlate events together a bit
better.  Again, I'd rather worry about disk space than drop log entries that
might've been a bit noisy at one point but end up being useful down the road.
Elasticsearch has a fairly rich language for querying the database, so I'd
imagine it's possible to come up with an expression strong enough to get
exactly what you need.

Down the road, my wish is to have logstash pull certain metrics out of events,
and throw them over to graphite for visualization (this is already implemented
in logstash 1.0.16, but there's a few showstopping bugs that prevented me from
keeping 1.0.16 installed.)

Anyway, log monitoring has advanced quite a bit over the past few years in OSS
software - this solution, while more complex to setup than something like
Splunk, also costs absolutely nothing in upfront costs, and covers much of the
same ground as Splunk.  This is also a way better way to view logs than using
tools such as swatch or logwatch, where you do your filtering right at the
source, rather than gather as much as possible and filter at the tail end of
your pipeline (where you're more likely to have a bit better judgment as to
whether those INFO logs were spewing a bunch of crap or not.)
