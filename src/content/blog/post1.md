---
title: 'Building a Serverless Discord Bot with Azure Functions'
description: 'Using Azure Functions to create a cheap, Serverless discord bot'
pubDate: 'Mar 23 2024'
heroImage: '/post_img.webp'
tags: ['Azure', 'Serverless', 'Discord']
---

# Building a Serverless Discord Bot with Azure Functions

Discord Slash Commands are a way to interact with bots on Discord. They provide a user-friendly interface for bot interactions, allowing users to execute commands directly from the chat input box by typing `/` followed by the command name. Today we'll look at how to use [Azure Functions](https://azure.microsoft.com/en-us/products/functions/) to create a Discord slash-command bot

## Azure Functions Overview

Azure Functions are a serverless computing service provided by Microsoft as part of the Azure cloud platform. It allows developers to write and execute code in response to a variety of events without the need to provision or manage servers. They are particularly well suited for event driven problems, IE "something (an event) happens, this is the code I run when it does (the function)"
