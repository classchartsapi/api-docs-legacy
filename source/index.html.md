---
title: Classcharts API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - typescript

toc_footers:
  - <a href='https://github.com/Classcharts-API/api-docs'>View Repo on Github</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Unofficial Classcharts API documentation
---

# Introduction

Welcome to the *unofficial* Classcharts API documentation. These docs aim to provide an in depth guide to using the API manually, or with our JS client libary. If you have any issues with these docs, please create a new [issue](https://github.com/Classcharts-API/api-docs/issues) by visiting our Github page.

# Basics
## Authentication
### Student Endpoint

```typescript
import { ClasschartsStudentClient } from 'classcharts-api'
const studentClient = new ClasschartsStudentClient('12astrut','12/20/2000')

await studentClient.login()
```
```shell
curl -X POST "https://www.classcharts.com/student/login" \
   -H 'Content-Type: application/x-www-form-urlencoded' \
   -d '_method=POST&code=12GFKDQY&dob=20/1/2000' \
   -sD - --output /dev/null

# In the response your (short lived) auth token will be in a JSON object called
# student_session_credentials returned from a set-cookie header:

# set-cookie: student_session_credentials={"remember_me":false,"session_id":"a77dshds3jsd8sdfh2k3"};
```
To authenticate with the student API, you need to provide your classcharts code and date of birth. Classcharts returns a short lived auth token in a cookie, this means you need to look for a header called `set-cookie` setting a cookie called `student_session_credentials`. This header contains your `session_id`, which is required to obtain your auth token. When using this token, call the [student info](#student-info) endpoint in which the `session_id` is your new auth token.
#### In Short:
1. Login using the above endpoint
2. Use the `session_id` from the `set-cookie` header in your authentication header
3. Call the [student info](#student-info) and use the token in `session_id` as your new authentication token in all future requests
An implemention can be found in the [JS Client Library](https://github.com/classchartsapi/classcharts-api-js/blob/a192c0f799c7891dfe16535e084b658ef6425511/src/studentClient.ts#L31)
### Parent API
```typescript
import { ClasschartsParentClient } from 'classcharts-api'
const parentClient = new ClasschartsParentClient('bob@example.com','Passw0rd')

await parentClient.login()
```
```shell
curl -X POST "https://www.classcharts.com/parent/login" \
   -H 'Content-Type: application/x-www-form-urlencoded' \
   -d '_method=POST&email=bob@example.com&password=Passw0rd&logintype=existing' \
   -sD - --output /dev/null

# In the response your auth token will be in a JSON object called
# student_session_credentials returned from a set-cookie header:

# set-cookie: student_session_credentials={"remember_me":false,"session_id":"a77dshds3jsd8sdfh2k3};
```
To authenticate with the parent API you need to provide your email address and password. The auth token is obtained the same way as the above student API.
## Requesting Data
With each request you must specify your `authorization` token, and optionally your student ID in the URL. However, it is **required** to specify the student ID when using the parent API, as Classcharts needs to know which student you would like to get data for.

# Student API
The base URL for the student API is `https://www.classcharts.com/apiv2student`  

## Student Info
```typescript
const data = await studentClient.getStudentInfo()
console.log(data)
```

```shell
curl "https://www.classcharts.com/apiv2student/ping"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```

`GET https://www.classcharts.com/apiv2student/ping` 

Gets basic information about the logged in student and what access they have.

## Get Activity
```typescript
const data = await studentClient.getActivity()
console.log(data)

const specificData = await studentClient.getActivity({
  from: "2000-12-20",
  to: "2022-03-20",
  last_id: 4563278
})
console.log(specificData)

// Our javascript libary provides a helper function to return all the activity data, without needing to use last_id
// See the note on the left
const fullSpecificData = await studentClient.getFullActivity({
  from: "2000-12-20",
  to: "2022-03-20",
})
console.log(fullSpecificData)
```
```shell
curl "https://www.classcharts.com/apiv2student/activity/2339528"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'

curl "https://www.classcharts.com/apiv2student/activity/2339528?from=2000-12-20&to=2022-03-20&last_id=4563278"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/activity/[userId]` 
 
This endpoint gets the latest behaviour points for the logged in user. Optional `from` and `to` fields can be used to scope the request to specific dates. The `last_id` is used by Classcharts for paginiation, and can be used to get results after a specific behaviour point.

Parameter | Required | Description
--------- | -------- | -----------
from | false | From date
to | false | To Date
<aside class="notice">
    The <code>from</code> field does not work as you would expect, since this endpoint is meant for pagination on the home page, Classcharts only returns a 50 points from the <code>to</code> field. To get more points you will need to make use of the <code>last_id</code> field. Our <a href="https://github.com/Classcharts-API/js">javascript wrapper</a> provides a <code>client.getFullActivity()</code> helper function to do this for you.
</aside>

## Get Behaviour

```typescript
const data = await studentClient.getBehaviour()
console.log(data)

const specificData = await studentClient.getBehaviour({
  from: '2000-12-20',
  to: '2022-03-20'
})
console.log(specificData)
```
```shell
curl "https://www.classcharts.com/apiv2student/behaviour/2339528"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'

curl "https://www.classcharts.com/apiv2student/behaviour/2339528?from=2000-12-20&to=2022-03-20"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/behaviour/[userId]` 
 
This endpoint returns basic statistics on how many of each type of behaviour point the logged in student has recieved. Optional `from` and `to` fields can be used to get statistics between two dates.

Parameter | Required | Description
--------- | -------- | -----------
from | false | From date
to | false | To Date
## Get Homeworks

```typescript
const data = await studentClient.listHomeworks()
console.log(data)

const specificData = await studentClient.listHomeworks({
  displayDate: 'due_date', // Can be due_date or issue_date, defaults to due_date
  from: '2000-12-20',
  to: '2022-03-20'
})
console.log(specificData)
```
```shell
curl "https://www.classcharts.com/apiv2student/homeworks/2339528"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'

# display_date can be due_date or issue_date
curl "https://www.classcharts.com/apiv2student/homeworks/2339528?display_date=due_date&from=2000-12-20&to=2022-03-20"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/homeworks/[userId]` 
 
This endpoint returns homeworks the logged in user has. Optional `from` and `to` fields can be used to scope the request. The `display_date` field is used to either find homeworks by due date (`due_date`) or issue date (`issue_date`).

Parameter | Required | Description
--------- | -------- | -----------
from | false | From date
to | false | To Date
display_date | false | How to sort the homeworks (`due_date` or `issue_date`)
## Get Timetable
```typescript
const data = await studentClient.getLessons({
  date: '2022-03-20' // Date to get the timetable for must be provided
})
console.log(data)
```
```shell
# Date to get the timetable for must be provided
curl "https://www.classcharts.com/apiv2student/timetable/2339528?date=2022-03-20"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/timetable/[userId]` 
 
This endpoint gets the logged in users timetable for a specific day. The required `date` field specifies which day to get the timetable for.

Parameter | Required | Description
--------- | -------- | -----------
date | true | Date to get the timetable for
## Get Badges
```typescript
const data = await studentClient.getBadges()
console.log(data)
```
```shell
curl "https://www.classcharts.com/apiv2student/eventbadges/2339528"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/eventbadges/[userId]` 
 
This endpoint returns any badges the logged in user has, along with the behaviour point that triggered it.

## Get Announcements
```typescript
const data = await studentClient.listAnnouncements()
console.log(data)
```
```shell
curl "https://www.classcharts.com/apiv2student/announcements/2339528"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/announcements/[userId]` 
 
This endpoint returns the announcements for the logged in user sent by the school.

## Get Detentions
```typescript
const data = await studentClient.getDetentions()
console.log(data)
```
```shell
curl "https://www.classcharts.com/apiv2student/detentions/2339528"  \
  -H 'Authorization: Basic 5vf2v7n5uk9jftrxaarrik39vk6yjm48'
```
`GET https://www.classcharts.com/apiv2student/detentions/[userId]` 
 
This endpoint returns the detentions the logged in user has.

# Parent API
The base URL for the parent API is `https://www.classcharts.com/apiv2parent`.  
The parent API is identical to the [student API](#student-api), except that the data returned is based on the student ID which is passed via each request and the [get pupils](#get-pupils) endpoint.

## Get Pupils
```typescript
const data = await parentClient.getPupils()
console.log(data)
```
```shell
curl "https://www.classcharts.com/apiv2parent/pupils"  \
  -H 'Authorization: Basic eadyjtgk7fnnvvqxuadmmdr5aibntaaf'
```
`GET https://www.classcharts.com/apiv2parent/pupils`  

This endpoint returns an array of the pupils the logged in parent has access to, the response is very similar to the [student info](#student-info) endpoint, aside from returning a couple more fields.
