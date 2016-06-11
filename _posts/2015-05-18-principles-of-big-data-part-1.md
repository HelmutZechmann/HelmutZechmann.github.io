---
layout: post
title: Principles Of Big Data - Part 1&#58; Your Input Data Is Immutable
tags: [big data]
---



Data storage systems are used to store information. This information may change in the course of time. You have two basic options to reflect these changes in your data storage systems:


1. Update the information
1. Store the new information in addition to the existing information

Consider the following example:

In a social network a user has a list of followers. This list may be modified by two events:

* *follow event* - A new follower is added to the list of followers
* *unfollow event* - A follower chooses to unfollow the user.

One possibility to store this information is to always store and update a list of current followers for each user. Each time a new follower is added or removed you update this list in your storage system.
The second possibility is to store all follow and unfollow events. The current list of followers for a user is derived from this information.


<table>
    <tr>
        <th>scenario</th>
        <th>solution one</th>
        <th>solution tow</th>
    </tr>
    <tr>
        <td>Get the current list of followers for the user Arthur.</td>
        <td>Read the list of current followers.</td>
        <td>Derive the list of followers from the sequence of follow and unfollow events.</td>
    </tr>
    <tr>
        <td>Get the number of followers for the user Arthur.</td>
        <td>Compute the length of the list of current followers.</td>
        <td>Derive the number of followers from the sequence of follow and unfollow events.</td>
    </tr>
    <tr>
        <td>Get all users that have been following Arthur two years ago.</td>
        <td>-</td>
        <td>Derive the list of followers from the sequence of follow and unfollow events</td>
    </tr>
    <tr>
        <td>Get a list of all users the user Ford has been following on 2000-01-01.</td>
        <td>-</td>
        <td>Derive the list of followed users from the sequence of follow and unfollow events.</td>
    </tr>
</table>



As you can see solution one offers a simple and efficient solution for answering the questions you may have had in mind while implementing the solution. But there are many possible questions you cannot answer with this data model.

Solutions two requires much more storage and also additional computation efforts to answer simple questions. This is a high price to pay, but you get a great reward: Since you do not lose information due to data updates you have the possibilites to answer that arise later in time. Questions that you did not even think of when you where implementing your application.

Solution two provides another big advantage: Since you never update your raw data the danger of data corruption due to an application error is much less!

