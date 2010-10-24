--- 
layout: post
title: Thoughts on game server fault tolerance
post_id: "101"
categories:
- Fault Tolerance
- Game Server Administration
- Games
- State Preservation
- Systems Administration
---
There has been several efforts to address the issue of ensuring fault tolerance for game servers; one approach is to introduce <a href="http://www.cs.colorado.edu/~mishras/research/ft_tcp.html">a new protocol based on TCP</a> that has extra checks to ensure fault tolerance.  <a href="http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.98.8585&rep=rep1&type=pdf">Another approach</a> is to section off the game world and structure it so that there are several zones, with each zone maintained by a different authority.

However, let's consider FPS game servers: the game players expect low latency for each action that they perform in the server, no noticeable network degradation when they're on one side of the map or another, and they expect the server hardware to not choke when the server is at full capacity.  Further, most FPS game server binaries <em>do not have application level fault tolerance built in</em>.  Therefore, we want to look at the first solution, which is to use a modified TCP protocol on each of the servers that we want to use, and see if we can provide fault tolerance without editing any of the client software nor the server software.

The major challenge (that I can see so far) is the following: we must preserve the game state between all of the participating game servers.  If one server goes down - assuming that our modified protocol migrates everyone perfectly to the backup server - we need to have the backup game server have the entire game environment replicated, so that it doesn't disrupt gameplay.  One way we could possibly do this is to have a machine that handles the TCP connections in front, and multicasts that traffic to both game servers.  Thus, the players are in fact playing on both game servers at the same time.

<a href="http://www.redbluemagenta.com/wp-root/wp-content/uploads/2010/04/fig1_gameserver.gif"><img src="http://www.redbluemagenta.com/wp-root/wp-content/uploads/2010/04/fig1_gameserver-300x300.gif" alt="" title="Figure 1, &quot;Game Server Fault Tolerance&quot;" width="300" height="300" /></a>

However, we cannot guarantee that the game state is in fact equivalent on both machines; this would be even more troublesome if there was an element of AI in the game, which we can safely assume will do different actions on either server.

Anyway, that's where I'm at right now so far.  I'll have to do a bit more research and see if ST-TCP is a feasible option (I'm also looking at another solution, called <a href="http://userweb.cs.utexas.edu/users/lorenzo/lft.html">FT-TCP</a>.)  Preserving the game state, however, is the biggest obstacle to tackle.
