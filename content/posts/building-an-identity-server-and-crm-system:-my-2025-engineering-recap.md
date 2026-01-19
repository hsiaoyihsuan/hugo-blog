---
title: "Building an Identity Server and CRM System: My 2025 Engineering Recap"
date: 2026-01-05
categories:

tags:

thumbnailImagePosition: left
thumbnailImage: images/2025-recap.png
---

![4 OAuth2 Grant Types](/images/2025-recap.png)

---

## Content

One of the biggest milestones for me in 2025 was seeing our company’s new repositories go live in May—and run smoothly throughout the rest of the year: the **CRM (Customer Relationship Management) service** and the **Identity Server**. After multiple iterations, prototypes, and rounds of internal feedback, both systems finally went into production. I was responsible for building both the Identity Server and the CRM service. With this setup, users can log in using different social media accounts and earn reward points when shopping in physical stores, through our mobile app, or via the online ordering website.

There’s an interesting story behind the Identity Server. At the beginning, we relied on **AWS Cognito**. When the number of stores and users was still small, the cost stayed within the free tier. But as our user base grew, Cognito started charging for each interaction, and the cost increased quickly. To reduce expenses and gain more control, we decided to build our own Identity Server.

Before this project, I only knew OAuth2 at a very basic level. This time, I had to build a service that could fully replace Cognito. The new Identity Server needed to support logins via Google, Apple, and other OIDC providers, while also issuing our own tokens. On top of that, it had to fully comply with OAuth2 and OpenID Connect so existing systems wouldn’t need to change anything. Back then, I had barely even heard of OIDC. I spent a lot of time reading protocol specifications, and carefully implementing most of the details. It was my first real deep dive into protocol documentation—painful at times, but also very rewarding. I also learned a lot by studying how a large-scale service like Cognito is designed. Now, I’m comfortable with these concepts and can explain them clearly to my teammates whenever questions come up.

The CRM service brought a different set of challenges and lessons. As the project evolved, the system design had to stay flexible enough to handle new requirements and changes. For example, reward points were initially calculated directly from orders, but later many new scenarios appeared. Points might come from completing specific tasks, compensating customers for certain issues, or other business cases. This meant the database schema and application logic couldn’t be tightly coupled. In some cases, denormalization was necessary, and design patterns like the strategy pattern had to be introduced.

As the system kept evolving, deploying new features to production became more challenging. We had to make sure APIs stayed backward compatible, and sometimes existing data needed to be migrated carefully. These moments really reminded me how important it is to design systems that can grow and change without breaking everything.

Every time the product manager walked into our office, we all gave him that look and laughed—because it usually meant new wishes from users were on the way, and everything might change again. 😄


---

## Conclusion

I graduated from Civil Engineering, and the software industry is very different from traditional engineering fields. Things move fast, requirements change often, and staying flexible and agile is essential. Even though 2025 wasn’t always easy, I learned a lot from these challenges. I hope to keep growing, learning, and improving myself in 2026.

Wish you all a great new year,

**Hsuan**
