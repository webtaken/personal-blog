---
layout: ../../layouts/BlogPostLayout.astro
id: post-2
title: "Your personal coding interview coach (AI agent)"
pubDate: 2023-11-19T14:52:00Z
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

In this post as Indie Creators, we will ride the wave to gather inspiration for potential use cases, fostering the creation of new side projects through this emerging technology. So our use case will be to create a **personal coding interview coach** that will help _**Ryan Gosling:** a junior software developer that is preparing himself for its next coding interview_ to get a job at an important software company.

<h1 id="creating-the-assistant">Creating the assistant <a href="#creating-the-assistant">#</a></h1>

First we'll need to create a new assistant. You can create one from the new [Assistants playground](https://platform.openai.com/playground?mode=assistant) in OpenAI's platform or using the new OpenAI's [Assistants API](https://platform.openai.com/docs/api-reference/assistants). We'll use the OpenAI Assistants API using the official [python SDK](https://github.com/openai/openai-python).  
So here is the code for creating our assistant, take attention in the initial instruction.

```python
instructions = """You are an experienced coding interview coach that will help a junior software
developer for an interview. Your main tasks will be:
- Give problems on most common topics in coding interviews and return some examples.
- When asked to evaluate a solution (in any programming language) for a problem, evaluate the
  correctness of the solution running the code over a sample of inputs."""

# Creating the assistant
assistant = client.beta.assistants.create(
    name="Personal Coding Interview Coach",
    instructions=instructions,
    model="gpt-3.5-turbo",
    tools=[{"type": "code_interpreter"}]
)
```

You will see your new assistant created on the [OpenAI's platform](https://platform.openai.com/assistants).  
The initial instruction is the behavior of our Assistant. We are telling it that its existence reason is coaching a junior dev in coding interviews. We also indicated it some tasks.

<h1 id="starting-chat">Starting a new chat with the assistant <a href="#starting-chat">#</a></h1>

In Assistants API a conversation is called a **thread**. The next step is to create a new thread to chat with our assistant.

```python
thread = client.beta.threads.create()
print(thread)
# Output: Thread(id='thread_JBmhDZmLqjVOZf09h9YbwpN1', created_at=1700410886, metadata={}, object='thread')
```

We created a new conversation, the sdk will return thread's properties, the most important information is the **thread id** on which we'll push messages.

> **Note:** A thread handles automatically the window size, so you don't have to worry about how many messages you send to it. OpenAI will fit automatically the entire chat with the context window of your selected model.

Now we'll push a message to the new chat, to do this we have to [create a new message](https://platform.openai.com/docs/api-reference/messages/createMessage) and add it to the thread.

```python
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Give me an easy problem with arrays"
)
print(message.json())
# Output: {"id": "msg_m79VrxSgSiSNcFJJXD3Ort8Q", "assistant_id": null, "content": [{"text": {"annotations": [], "value": "Give me an easy problem with arrays"}, "type": "text"}], "created_at": 1700412050, "file_ids": [], "metadata": {}, "object": "thread.message", "role": "user", "run_id": null, "thread_id": "thread_JBmhDZmLqjVOZf09h9YbwpN1"}
```

The message was added to the thread but the assistant hasn't answered anything yet we need to run the assistant.

<h1 id="chatting">Chatting with the assistant <a href="#chatting">#</a></h1>

To receive an answer from the assistant we will run the assistant explicitly. So we create a new `Run` object.

```python
run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="Please address the user as Ryan Gosling. The user has a premium account."
)
print(run)
# Output: Run(id='run_A9EMAXzrgUgSSJkyuVKaF59w', assistant_id='asst_YVgo81fA4z8uZSCeKGhpABtz', cancelled_at=None, completed_at=None, created_at=1700412670, expires_at=1700413270, failed_at=None, file_ids=[], instructions='Please address the user as Ryan Gosling. The user has a premium account.', last_error=None, metadata={}, model='gpt-3.5-turbo', object='thread.run', required_action=None, started_at=None, status='queued', thread_id='thread_JBmhDZmLqjVOZf09h9YbwpN1', tools=[ToolAssistantToolsCode(type='code_interpreter')])
```

We pass the **thread id** and the **assistant id**, you can pass as well additional instructions to have a more personalized response. In this case we say that our user is Ryan Gosling our junior dev chad that has a premium account in our side project.  
By default the Run object starts with the `queue` status. You can periodically retrieve the Run to check on its status to see if it has moved to `completed`.

```python
run = client.beta.threads.runs.retrieve(
  thread_id=thread.id,
  run_id=run.id
)
print(run)
# Output: Run(id='run_A9EMAXzrgUgSSJkyuVKaF59w', assistant_id='asst_YVgo81fA4z8uZSCeKGhpABtz', cancelled_at=None, completed_at=1700412674, created_at=1700412670, expires_at=None, failed_at=None, file_ids=[], instructions='Please address the user as Ryan Gosling. The user has a premium account.', last_error=None, metadata={}, model='gpt-3.5-turbo', object='thread.run', required_action=None, started_at=1700412670, status='completed', thread_id='thread_JBmhDZmLqjVOZf09h9YbwpN1', tools=[ToolAssistantToolsCode(type='code_interpreter')])

```

After retrieving the Run object and check that it passes to `completed` status we can retrieve all the messages in the thread.

```python
messages = client.beta.threads.messages.list(
  thread_id=thread.id
)
# Printing from oldest messages to the newest ones
for message in reversed(messages.data):
    print(f"{message.role}: {message.content[0].text.value}")

# Output:
# user: Give me an easy problem with arrays
# user: Give me an easy problem with arrays
# assistant: Sure, how about this:

# Problem: Find the maximum element in an array.

# Write a function `find_max(arr)` that takes in an array `arr` as input and returns the maximum element in the array.

# For example, given the input array `[3, 9, 2, 5, 1]`, the function should return `9` since `9` is the largest element in the array.

# To solve this problem, you can initialize a variable `max_num` to the first element of the array. Then, iterate through the remaining elements of the array and update `max_num` if you find a larger number.

# Please go ahead and write the code to solve this problem.
```

We can see that our assistant has answered us with a very easy problem find the maximum number
in an array of numbers.

<h1 id="solving-the-problem">Solving the problem <a href="#solving-the-problem">#</a></h1>

So now we will send our solution to the assistant. The solution will be in python, and the assistant with its integrated [code interpreter tool](https://platform.openai.com/docs/assistants/tools/code-interpreter) will evaluate if our solution is correct.

```python
answer = """Here is my answer in python code is it right?:
`
def find_max(arr):
  if len(arr) == 0:
    return
  maximum = arr[0]
  for num in arr:
    if num > maximum:
      maximum = num
  return maximum
`
"""
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content=answer
)

run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="Please evaluate this code, test it with corner cases as well."
)
```

> Note: I passed the code in text but the code interpreter tool allows you to upload a file to evaluate your answer. We'll do it in future posts.

We wait until the Run status passes to `completed` and then check the assistant last response.

```python
messages = client.beta.threads.messages.list(
  thread_id=thread.id
)

# Printing the last response from the assistant
print(f"{messages.data[0].role}: {messages.data[0].content[0].text.value}")

# Output:
# assistant: Your code passed all the test cases, including corner cases. Well done! The function correctly finds the maximum element in the array.

# Here are the outputs for the test cases:
# - Test Case 1: `9`
# - Test Case 2: `None`
# - Test Case 3: `-1`
# - Test Case 4: `8`
# - Test Case 5: `7`

# Is there anything else I can help you with?

```

So our code passed all the tests and our answer seems to be correct.

<h1 id="code">Source Code <a href="#code">#</a></h1>

You can find the source code for this example on: [source code](https://github.com/webtaken/AI-scripts/blob/main/agents/coding-interview-coach.ipynb).

<h1 id="side-project-idea">Side project idea <a href="#side-project-idea">#</a></h1>

So we have built a basic coding interview coach that needs to be improved a lot to become a decent competition to existing products such as [Leetcode](https://leetcode.com/) for example.  
The basic idea for now is to create a newsletter sending an easy-to-intermediate problem each day to our users by email. With a basic monthly subscription plan we can train a junior dev to tackle its next coding interview. The challenge here is answer the argument: **And what's the difference with other similar sites man 🧐?, there are a lot even for free** (we have to solve that issue).  
Another idea could be to do an assistant not for coding interviews instead a coach that helps you learn any programming language. We could copy the site [exercism.org](https://exercism.org/)
and create a complete roadmap for any programming language.  
Anyway I think I'll go for the newsletter if I have time...  
Start easy and iterate overtime to add more features, that's the best approach. Hope this post was helpful to you, don't forget to copy the idea or create a new one. The key is not to lose time 🦾.
