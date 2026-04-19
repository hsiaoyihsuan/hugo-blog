---
title: "My First Real Experience Using AI to Refactor Legacy Code"
date: 2026-04-19
categories:

tags:
- AI

thumbnailImagePosition: left
thumbnailImage: images/2026_ai_refactor.png
---

![My First Real Experience Using AI to Refactor Legacy Code](/images/2026_ai_refactor.png)

---


## Introduction

In February 2026, Anthropic published a [blog post](https://claude.com/blog/how-ai-helps-break-cost-barrier-cobol-modernization) about using Claude Code to modernize legacy COBOL systems. After that, many people said the technical moat was disappearing, human experts would be replaced, and even IBM's stock price dropped sharply.

Personally, I think many people didn't fully read the article or understand the real meaning behind it. The value of AI tools in legacy refactoring is not to replace engineers. It's about helping engineers solve problems that were previously slow, painful, or difficult to tackle.

I recently had a similar experience using Codex (not Claude Code, but a similar idea) to refactor legacy code at work. My biggest takeaway: AI is powerful, but it still needs human judgment, domain knowledge, and clear direction.

## Content

Here's the background. Our company has a Node.js microservice for **Discount**, which supports multiple products such as our online ordering system and POS system. It's much more complex than it might sound. Different discount rules can interact with products, stores, other rules, and various constraints.

Since I'm developing a new **Coupon** feature in 2026, it needs to integrate with the existing Discount feature. However, they were in different repositories. That meant I needed to migrate the Discount service into our new CRM repository built with NestJS.

At first, I didn't realize how complex the Discount feature really was. I had been using OpenAI Codex with the GPT-5.3-Codex model for about a month, and it had already significantly accelerated my development workflow. I wanted to use this refactor as a real test of AI's ability to handle legacy code.

**Well... things didn't go as expected.**

The AI kept going back and forth, generating code that looked reasonable on the surface but missed important details everywhere. Even when I provided context and carefully managed the context window, it still struggled to fully understand the overall logic.

**The real issue wasn't that AI wasn't smart.**

The codebase itself was incomplete, some supporting documents were outdated or misleading, and much of the business logic only existed in people's heads. That's why refactoring real-world systems is far more complicated than the ideal "one-shot AI refactor" people sometimes imagine.

The legacy code had several problems:

1. API documentation was outdated, and some parts were even misleading.
2. Tests didn't fully cover the core features.
3. Poor readability: the repo was written in TypeScript, but it used a lot of `any` types. Database queries were written as raw SQL strings with little type safety.
4. Poor structure: the API layer, business logic, and database logic were tightly mixed together.
5. Important business rules were implicit and not clearly documented in either code or docs.

At that moment, I knew this was where human engineers still shine, but it also felt painful at the same time. It was a very conflicted feeling 😂.

To solve the problem, I spent almost a week clarifying the API docs and collecting business logic from teammates who had worked on the Discount feature before. I also gathered API test cases from the QA environment and checked the database directly. By doing that, I gradually understood how the feature really worked and finally had enough reliable context to work with AI effectively.

Once I had the right information, I started collaborating with AI again. My process looked like this:

1. Imported test data into my local database.
2. Rebuilt the schema with an ORM and defined clear types and structures.
3. Created a reusable skill that explained the core refactor context, so it could be reused across multiple AI sessions.
4. Built simple query endpoints to inspect data and verify types quickly.

At that point, AI became much more accurate and useful. Then I started the core migration work. I mainly used two AI windows:

- One focused on analyzing the legacy code, helping me understand the data flow and code boundaries.
- The other focused on the new NestJS repository, generating clean code from my prompts or pseudocode.

Why not use just one AI window? Because the legacy code could easily pollute the model with misleading information and bad architecture. Separating the roles worked much better.

By breaking the migration into smaller pieces and reviewing each step carefully, I was able to successfully migrate the core feature into the new repository.

After the migration, I finally had a much stronger codebase to work with. And once the foundation became solid, AI started accelerating development again.

This experience reminded me that successful AI-assisted refactoring is not about fancy prompts or tricks. It still depends on clear context, strong engineering judgment, and a solid codebase.

## Conclusion

My experience strongly aligned with Anthropic's article. We still need human experts to set priorities, understand business constraints, define testing standards, and validate the migrated system.

The recommended approach is:

1. Use AI to understand and document the legacy system.
2. Let engineers review the findings and create a migration plan.
3. Refactor **incrementally**, one component at a time.
4. Continuously test and validate so the risk stays manageable.

AI is still an excellent tool for engineers. The key is knowing when to step in, how to guide it, and how to keep the results aligned with your expectations.

This was a very interesting experience for me, and I hope this sharing can be helpful to you as well. If you have any thoughts or similar experiences, feel free to share them with me.

## References

- [Claude - How AI helps break the cost barrier to COBOL modernization](https://claude.com/blog/how-ai-helps-break-cost-barrier-cobol-modernization)
