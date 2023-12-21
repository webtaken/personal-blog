---
layout: ../../layouts/BlogPostLayout.astro
id: post-4
title: "Fast youtube video sentiment analysis with Go routines + Hugging Face 🤗"
pubDate: 2023-12-14T00:00:00Z
description: "Build a fast sentiment analyzer for youtube videos using go routines and hugging face models"
descriptionExtended: "In this post I'll show you the power of Go routines using them for fast sentiment analysis using hugging face public models."
author: "Saul Rojas"
authorURL: "https://www.linkedin.com/in/saul-rojas-6885b1188/"
image:
  url: "https://images.openai.com/blob/2014517b-1a80-4b62-bbb6-caa490f69299/introducing-gpts.png?trim=0,0,0,0&width=1400"
  alt: "OpenAI GPTs"
tags:
  [
    "blogging",
    "AI",
    "Hugging Face",
    "Go",
    "Go routines",
    "concurrency",
    "Youtube API",
  ]
---

Welcome to a new blog post dear visitor – In this post, we'll code a very interesting tool to analyze
the comments of a youtube video. Using the power of go routines, hugging face models and Youtube data API we'll code a basic sentiment analyzer that will classify the comments of any youtube video on internet.

Nowadays with the hype of AI almost anyone can combine different AI tasks (`Text-to-Image`, `Image to-Text`, `Natural Language Processing Tasks`, etc) to many sources of information and apps: text editors, videos, images, personal calendars, etc etc.
All this process can take too much time in many cases, that's why we need fast programming languages that let us take an input and process it as fast as possible. Go has a great advantage with its focus in concurrency, apart from being a compiled language.

So let's start building our sentiment analyzer for our youtube videos, all the source code is stored here.

> Source code: [github.com](https://github.com).

<h1 id="setup">Setup <a href="#setup">#</a></h1>

Our tech stack will consist of three tools **Go (programming language)**, **Youtube Data API (v3)** `token` to extract the comments from the videos and **Hugging Face Inference models**. Here are the steps to get all of them, once you have all set up come back to the tutorial:

- To install Go in your computer you can go to the official documentation: <a href="https://go.dev/doc/install" target="_blank">here</a>.
- To get a Youtube Data API token see this <a href="https://blog.hubspot.com/website/how-to-get-youtube-api-key" target="_blank">tutorial</a> on how to get one.
- To get a Hugging Face Inference API key you can go to the <a href="https://huggingface.co/docs/api-inference/quicktour" target="_blank">official documentation</a> or if you want to know more about this service check my <a href="./hugging-face-low-cost-models" target="_blank">blog post</a>.

Once you have all this set up let's create the directory for our project **_use wsl if you use windows_**.

```
mkdir sentiment-analysis
cd sentiment-analysis
touch sentiment-analysis
go init sentiment-analysis
```

Then we'll install the sdk for Youtube API and that's all that we need to start our project.

```
go get google.golang.org/api/youtube/v3
```

<h1 id="getting-started">Getting started <a href="#getting-started">#</a></h1>

The first step we'll do is to retrieve basic data from a video using the Youtube API. Our program will receive two parameters a YT video ID, those that appear in the query param `?v=<videoId>` every time we watch a video in our browser e.g `https://www.youtube.com/watch?v=446E-r0rXHI&ab_channel=Fireshi`, and the number of comments we want to analyze.

> Note: The comments we'll analyze will be the top comments of a video not replies.

So our initial `main` function starts like this:

```go
var youtubeAPIToken = os.Getenv("YOUTUBE_API_TOKEN")
var Service *youtube.Service

func init() {
	ctx := context.Background()
	service, err := youtube.NewService(ctx, option.WithAPIKey(youtubeAPIToken))
	if err != nil {
		log.Fatalf("%v\n", err)
	}
	Service = service
}

func main() {
	var videoId string
	var numComments int
	flag.StringVar(&videoId, "videoId", "", "The id of a youtube video e.g. yyUHQIec83I")
	flag.IntVar(&numComments, "numComments", 100, "The number of comments to analyze (default: 100)")
	flag.Parse()

	if numComments <= 0 {
		log.Fatal("numComments flag must be a positive number")
	}

	videoData, err := getVideoData(videoId)

	if err != nil {
		log.Fatal(err.Error())
	}

	printYoutubeVideoData(videoData)
}
```

We define the `Service` variable of type `*youtube.Service` that is the client to do calls to the Youtube API, we initialize it on the `init` function. Notice the global variable `youtubeAPIToken` that retrieves the env var **YOUTUBE_API_TOKEN**. Also we use the built-in `flag` package in Go to parse our command parameters `videoId` and `numComments` (by default 100). We do some validations like the video id must not be empty and number of comments must be positive and not be zero.  
The function `getVideoData()` and `printYoutubeVideoData()` will be used to get and show basic data of the video we'll analyze.

```go
func getVideoData(videoId string) (*youtube.VideoListResponse, error) {
	var part = []string{"snippet", "contentDetails", "statistics"}
	call := Service.Videos.List(part)
	call.Id(videoId)
	response, err := call.Do()
	if err != nil {
		return nil, err
	}
	if len(response.Items) == 0 {
		return nil, fmt.Errorf("video not found")
	}
	return response, nil
}

func printYoutubeVideoData(videoData *youtube.VideoListResponse) {
	fmt.Printf("Video Data\n")
	fmt.Printf("Title: %s\n", videoData.Items[0].Snippet.Title)
	fmt.Printf("-------------------------------------------------\n")
	fmt.Printf("Description: %s\n", videoData.Items[0].Snippet.Description)
	fmt.Printf("-------------------------------------------------\n")
	fmt.Printf("Likes: %d\n", videoData.Items[0].Statistics.LikeCount)
	fmt.Printf("Comments Count: %d\n", videoData.Items[0].Statistics.CommentCount)
	fmt.Printf("Views Count: %d\n", videoData.Items[0].Statistics.ViewCount)
}
```

To start we'll get basic data from a video like its: Title, Description, Number of Likes, Number of Comments (top comments + replies), and the Number of Views. Let's do a try with this video (<a href="https://www.youtube.com/watch?v=446E-r0rXHI&ab_channel=Fireship" target="_blank">Go in 100 seconds</a>) with id **446E-r0rXHI**.  
To get the results we execute the command:

```
YOUTUBE_API_TOKEN=<your-youtube-api-token> go run main.go -videoId 446E-r0rXHI
```

And the output is this:

```
Video Data
Title: Go in 100 Seconds
-------------------------------------------------
Description: Learn the basics of the Go Programming Language. Go (not Golang) was developed at Google as a modern version of C for high-performance server-side applications. https://fireship.io/lessons/learn-go-in-100-lines/

#programming #go #100SecondsOfCode

🔗 Resources

Go in 100 Lines https://fireship.io/lessons/learn-go-in-100-lines/
Go Docs https://golang.org/doc/

🔥 Get More Content - Upgrade to PRO

Upgrade to Fireship PRO at https://fireship.io/pro
Use code lORhwXd2 for 25% off your first payment.

🎨 My Editor Settings

- Atom One Dark
- vscode-icons
- Fira Code Font

🔖 Topics Covered

- History of Go Development
- Programming Languages Invented by Ken Thompson
- Statically-typed Complied Languages
- Go Modules
-------------------------------------------------
Likes: 56076
Comments Count: 1106
Views Count: 1340139
```

So we have a basic script that let us know basic data from a youtube video. Let's continue.

<h1 id="sequential-analyzer">Sequential Analyzer <a href="#sequential-analyzer">#</a></h1>

Before coding our concurrent analyzer we'll start with the sequential one to have a grasp of the steps to take during the analysis.  
So the first step is to define the functions involved, we add the function `analyzeVideoSequential()`
at the last part of of the `main` function. The `analyzeVideoSequential()` function will receive the video id and the number of comments to analyze.

```go
func main() {

  ...

	fmt.Printf("\nAnalyzing video (Sequential method)...\n")
	startTime := time.Now()
	analyzeVideoSequential(videoId, numComments)
	fmt.Printf("Analysis took %.3f seconds to execute.\n", time.Since(startTime).Seconds())
}
```

The `analyzeVideoSequential()` function get the video comments and analyze with the Hugging Face
inference models a batch of comments. Let's break it down.

```go
func analyzeVideoSequential(videoId string, numComments int) {
	positive := Counter{
		Tag:     "Positive",
		Count:   0,
		Channel: make(chan int),
	}
	negative := Counter{
		Tag:     "Negative",
		Count:   0,
		Channel: make(chan int),
	}
	neutral := Counter{
		Tag:     "Neutral",
		Count:   0,
		Channel: make(chan int),
	}
	errors := Counter{
		Tag:     "Errors",
		Count:   0,
		Channel: make(chan int),
	}

	var part = []string{"id", "snippet"}
	nextPageToken := ""
	call := Service.CommentThreads.List(part)
	pageSize := BATCH // Returns a page of max BATCH size
	call.VideoId(videoId)
	call.MaxResults(int64(pageSize))
	commentsRetrieved := 0
	commentsRetrieveFailed := 0
	for commentsRetrieved < numComments {
		if nextPageToken != "" {
			call.PageToken(nextPageToken)
		}

		response, err := call.Do()
		if err != nil {
			commentsRetrieveFailed += pageSize
			continue
		}

		commentsToAnalyze := len(response.Items)
		if commentsRetrieved+commentsToAnalyze >= numComments {
			commentsToAnalyze = numComments - commentsRetrieved
		}
		commentsRetrieved += commentsToAnalyze

		batchAnalyzerSequential(response.Items[:commentsToAnalyze], &positive, &negative, &neutral, &errors)

		nextPageToken = response.NextPageToken
		if nextPageToken == "" {
			break
		}
	}

	// After all the process we show the results
	printAnalysisResults(&positive, &neutral, &negative, &errors, numComments)
}
```

The section with four struct variables of type `Counter` will be the counters for each category positive, negative, neutral or error (whenever Hugging Face fails analyzing the comment).

```go
positive := Counter{
		Tag:     "Positive",
		Count:   0,
		Channel: make(chan int),
}
negative := Counter{
  Tag:     "Negative",
  Count:   0,
  Channel: make(chan int),
}
neutral := Counter{
  Tag:     "Neutral",
  Count:   0,
  Channel: make(chan int),
}
errors := Counter{
  Tag:     "Errors",
  Count:   0,
  Channel: make(chan int),
}
```

The `Counter` struct is defined below, we have a tag, an integer that is a counter and a channel that will be used in the concurrent program to communicate the results:

```go
type Counter struct {
	Tag     string
	Count   int
	Channel chan int
}
```

The next section is the for loop where we get the youtube video comments by pages until we reach the number of comments specified in the function parameters. We name the variable `call` to be the service that will retrieve the comments from the video. Notice the constant `BATCH` int the `MaxResults()` method, this is used to define the page size for the response. Check out the <a href="https://developers.google.com/youtube/v3/docs/comments/list" target="_blank">documentation</a> about this method in the youtube data API docs:

```go
	...
	var part = []string{"id", "snippet"}
	nextPageToken := ""
	call := Service.CommentThreads.List(part)
	call.VideoId(videoId)
	call.MaxResults(int64(BATCH))
	commentsRetrieved := 0
	commentsRetrieveFailed := 0
	for commentsRetrieved < numComments {
		if nextPageToken != "" {
			call.PageToken(nextPageToken)
		}

		response, err := call.Do()
		if err != nil {
			commentsRetrieveFailed += pageSize
			continue
		}

		commentsToAnalyze := len(response.Items)
		if commentsRetrieved+commentsToAnalyze >= numComments {
			commentsToAnalyze = numComments - commentsRetrieved
		}
		commentsRetrieved += commentsToAnalyze

		batchAnalyzerSequential(response.Items[:commentsToAnalyze], &positive, &negative, &neutral, &errors)

		nextPageToken = response.NextPageToken
		if nextPageToken == "" {
			break
		}
	}
	...
```

When there are no more pages `if nextPageToken == "` we break the loop. Then we have the function `batchAnalyzerSequential()` that as the name suggest it will analyze a batch of comments extracted youtube. We send by reference the `&positive, &negative, &neutral, &errors` to update the counters.  
Since this is done sequentially we don't care about yet of data racing problem in concurrent programs.  
The function `batchAnalyzerSequential()` is defined below, we receive an slice of `youtube.CommentsThread` pointers and the four counters. On this function we'll do an api request to the Hugging Face Inference API (`Roberta` model) to get the sentiment classification for the batch of messages. Notice the `jsonData` variable that will store the request body that is build by the function `buildRequestBody()`.  
In case of any error, building the client or not receiving an OK status from the API, we sum up to the error counter the len of the comments we are processing. And in case the api request goes well
we'll unmarshall the json response to update the `positive, negative, neutral` counters.

```go
func batchAnalyzerSequential(
	comments []*youtube.CommentThread,
	positive *Counter,
	neutral *Counter,
	negative *Counter,
	errors *Counter,
) {
	modelEndpoint := "https://api-inference.huggingface.co/models/cardiffnlp/twitter-xlm-roberta-base-sentiment"
	var jsonData = []byte(buildRequestBody(comments))
	request, err := http.NewRequest("POST", modelEndpoint, bytes.NewBuffer(jsonData))
	if err != nil {
		errors.Count += len(comments)
		return
	}
	request.Header.Set("Authorization", fmt.Sprintf("Bearer %s", huggingFaceAPIToken))

	client := &http.Client{}
	response, err := client.Do(request)
	if err != nil {
		errors.Count += len(comments)
		return
	}

	defer response.Body.Close()

	if response.StatusCode == 200 {
		body, _ := io.ReadAll(response.Body)
		var sentimentAnalysisResponse []RobertaResponse

		err := json.Unmarshal(body, &sentimentAnalysisResponse)
		if err != nil {
			errors.Count += len(comments)
			return
		}
		handleModelResponseSequential(sentimentAnalysisResponse, positive, negative, neutral)
		return
	}
	errors.Count += len(comments)
}
```

The `buildRequestBody()` function defined below is in charge of building the json body received by the Hugging Face API
