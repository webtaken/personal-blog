---
layout: ../../layouts/BlogPostLayout.astro
id: post-2
title: "Your personal coding interview coach (AI agent)"
pubDate: 2023-11-16
description: "Build your personal programming interview coach AI agent with new OpenAI GPTs"
descriptionExtended: "In this post I'll show you the power of new OpenAI agents to build your own coding interview coach to prepare yourself to get a new job"
author: "Saul Rojas"
authorURL: "https://www.linkedin.com/in/saul-rojas-6885b1188/"
image:
  url: "https://docs.astro.build/assets/full-logo-light.png"
  alt: "AI agents post"
tags: ["blogging", "AI", "OpenAI"]
---

OpenAI has launched <a href="https://openai.com/blog/introducing-gpts" target="_blank">GPTs</a> its new feature that promise to change the game in how AI products are built today. This innovative offering promises to empower individuals globally, granting them the ability to effortlessly create their own personalized agents 🤖. This democratization of AI not only marks a significant shift in accessibility but also signals a departure from traditional models of development. With GPTs, OpenAI has opened the door for every person in the world to actively participate in shaping the future of artificial intelligence, fostering a new era of creativity, customization, and widespread engagement with this cutting-edge technology.

In this post, as Indie Creators, we will ride the wave to gather inspiration for potential use cases, fostering the creation of new side projects through this emerging technology. So our use case will be to create a **personal coding interview coach** that will help _**Ryan Gosling:** a junior software developer that is preparing himself for its next coding interview_ for an important software company.

<h1 id="creating-the-assistant">Creating the assistant <a href="#creating-the-assistant">🔗</a></h1>

First we'll need to create a new assistant. You can create one from the new [Assistants playground](https://platform.openai.com/playground?mode=assistant) in OpenAI's platform or using the new OpenAI's [Assistants API](https://platform.openai.com/docs/api-reference/assistants). We'll use the OpenAI Assistants API using the official [python SDK](https://github.com/openai/openai-python)
