---
layout: post
title:  "Tournify Discord Bot"
date:   2020-10-17 00:00:00
categories: tournify
tag: "tournify discord bot"
---

# What is Tournify?

In short, Tournify is a service aimed at unifying API from different tournament providers and keep users updated about games played as soon as they're over.
To learn more about Tournify, check [https://migwel.dev/tournify/2020/10/17/Tournify.html](https://migwel.dev/tournify/2020/10/17/Tournify.html).

# What is Tournify Discord Bot?

It's a Discord bot that works with Tournify. For example, you can ask the bot to subscribe to a specific tournament and it will post all results from that tournament into your discord group. That way, if some games are played offstream or if you cannot watch it, you can still be notified of the results as soon as they're published.

# Commands

The commands endpoints are available:

- `@TournifyBot subscribe tournamentUrl (player)` This command asks the bot to follow the tournament. As soon as a result is published, the bot will post it in your Discord group. If you are only interested in the results of a specific player, you can also specify it in the command. At the moment, the only way to follow multiple players is to execute the same command multiple time, changing the `(player)`. I'm planning on allowing to provide a list of players but it's not implemented yet
- `@TournifyBot participants tournamentUrl` The bot will return the list of participants at a tournament in format `prefix username`.

# Examples

- `@TournifyBot subscribe <https://smash.gg/tournament/genesis-6/events/melee-singles/overview`> The bot will post all results related to Genesis 6 Melee Singles, for all players
- `@TournifyBot participants <https://smash.gg/tournament/genesis-6/events/melee-singles/overview`> The bot returns all players participants at Genesis 6 Melee Singles
- `@TournifyBot subscribe <https://smash.gg/tournament/genesis-6/events/melee-singles/overview> C9 Mango` The post will post results related to Genesis 6 Melee Singles for C9 Mango

# Supported Tournament Hosts

Currently, the following tournament hosts are supported:

- [smash.gg](https://smash.gg/)
- [challonge.com](https://challonge.com/)

# How to invite Tournify Discord Bot

To invite the bot to your group, follow [this link](https://discordapp.com/oauth2/authorize?&client_id=221323100476801024&scope=bot&permissions=0)

# Getting in touch

If you have questions or suggestions, feel free to join the discord channel using [this link](https://discord.gg/D6GvMuR)

#Source code
If you are interested in how it's build and want to contribute, you can check the code on [Github](https://github.com/Migwel/tournify_discord_bot) 
