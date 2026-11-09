- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Power BI Copilot — Developer Readiness Presentation

1. Opening — Taking Over the Presentation

[Previous presenter completes the Copilot demonstration and hands over to you.]

“Thanks.

Alright, developers, this next part is for you.

We’ve just seen what Copilot can do from an end-user perspective.

Now, if you want your end users to get accurate, relevant, and meaningful responses from Copilot, there are three key things you need to focus on.

[Pause briefly.]

First, how you structure your data.

Second, how you enrich it with business context.

And third, how you prepare it for AI.

So, let’s walk through each of these and see how they contribute to a better Copilot experience.”

[Switch to Power BI Desktop.]

⸻

2. First Step — Semantic Model and Star Schema

[Open the semantic model in Model view.]

“So, the first thing is how we structure our semantic model.

Ideally, we should be leveraging our existing enterprise data products wherever available, as they provide a consistent and reusable data foundation.

And when we build our semantic model on top of that, it’s important to follow modeling best practices, particularly the star schema.

As you can see here, we have our fact table at the center, connected to our dimension tables.

The fact table contains our measurable business data, while the dimension tables provide additional context around that data.

For example, if we want to analyze transactions, the fact table contains the transaction information, while our dimensions might provide details such as customer, product, and date.

Now, why is this important for Copilot?

[Pause.]

Because Copilot uses the relationships, measures, and business logic defined in our semantic model to interpret questions.

If our model has ambiguous relationships, inconsistent calculations, or unnecessary complexity, Copilot may struggle to interpret what the user is asking.

A well-structured star schema helps reduce that ambiguity and provides a clearer foundation for analysis.

It also improves maintainability and can help with query performance.

And one important point here is that these aren’t practices we should follow only for Copilot.

These are standard Power BI modeling best practices that we should already be following.

Copilot simply makes the importance of having a well-designed semantic model even more visible.

So, that’s our first step: make sure the semantic model is properly structured before we start preparing it for AI.”

[Move to the next section.]

⸻

3. Second Step — Metadata Enrichment

[Select a table, column, or measure and open its Properties pane.]

“Now that we have our semantic model structured correctly, let’s move to the second step, which is metadata enrichment.

This is another important area when preparing our models for Copilot.

As developers, we’re familiar with our data. We know what our tables represent, what our columns mean, and how our measures are calculated.

But Copilot needs that business context as well.

Let me give you a simple example.

Let’s say we have a measure called ‘Active Customers’.

Now, what exactly does active mean?

Does it mean a customer who has completed a transaction in the last 30 days?

Or does it mean someone who currently holds an active account?

Those are two different business definitions.

And if we don’t provide that context, there’s a possibility that Copilot may not interpret the measure the way we intended.

So, this is where metadata enrichment comes into play.”

[Show the Description property of a measure or column.]

“Here, we can add meaningful descriptions to our tables, columns, and measures.

These descriptions should clearly explain what the data represents and include any important business definitions.

We should also use meaningful naming conventions, avoid unnecessary abbreviations, and make sure our measures have clear and consistent definitions.

And this becomes particularly important when our semantic models are being consumed by users across different lines of business.

Because what might be obvious to the team developing the model may not be obvious to someone using that data for the first time.

[Pause.]

So, if I were to summarize this step in one sentence:

Our semantic model defines how the data is connected, while metadata helps explain what that data actually means.

And that additional context helps Copilot interpret business questions more effectively.

That’s our second step.”

⸻

4. Third Step — Prep Data for AI

[Open the Prep data for AI experience.]

“Now, moving on to our third and final step: preparing our semantic model specifically for AI.

Once we have our model structured correctly and our metadata enriched, Power BI gives us additional capabilities through a feature called Prep data for AI.

This is where we can provide more focused guidance to Copilot on how to work with our data.

There are three capabilities I want to quickly walk through here.”

A. AI Data Schema

[Open the AI data schema or Simplify data schema section.]

“The first one is the AI data schema.

Now, in a typical enterprise semantic model, we might have a large number of tables, columns, and measures.

But not all of them are necessarily relevant when an end user asks Copilot a business question.

For example, we might have technical identifiers, intermediate calculations, or supporting fields that are important for the model itself but aren’t meaningful to the business user.

So, rather than having Copilot work with every available field, we can define a more focused set of business-relevant fields.

And that’s what the AI data schema helps us do.

[Show the selected fields.]

“Here, we can specify which tables and fields Copilot should prioritize for answering data questions.

This helps simplify the information Copilot needs to interpret and reduces unnecessary complexity.

So, the idea is straightforward: give Copilot the relevant business data it needs, rather than unnecessary technical details.”

B. Verified Answers

[Open the Verified Answers section and show a prepared example.]

“Next, we have Verified Answers.

This is a useful capability, especially when there are common business questions that users ask repeatedly.

Let’s say our business leaders frequently ask a question about the performance of a particular KPI.

And we already have a report visual that has been developed, reviewed, and validated to answer that question.

Instead of relying entirely on Copilot to generate a new answer each time, we can configure that existing visual as a verified answer.

We can also associate it with relevant questions or phrases.

So, when someone asks a matching question, Copilot can surface that predefined visual.

[Show the example.]

“This is valuable because it gives us a way to guide users toward an existing, validated business answer.

And particularly for important business metrics, that additional level of consistency can be very useful.”

C. AI Instructions

[Open the AI Instructions section.]

“And finally, we have AI Instructions.

This is where we can provide additional business-specific instructions to Copilot.

For example, within our organization, we may have certain terms or business definitions that are specific to a particular line of business.

We can use AI Instructions to explain those terms and guide Copilot toward the appropriate measures and business logic.

Let’s say users refer to a particular metric using a commonly understood business term, but the actual measure in our semantic model has a different name.

We can provide that context here.

We can also describe how certain business questions should be interpreted.

[Show the prepared instructions.]

“The important thing to understand is that these instructions don’t replace the underlying calculations or relationships in our semantic model.

They provide additional guidance to help Copilot understand our business terminology and expected interpretations.

So, to summarize Prep data for AI:

We have the AI data schema to focus on relevant fields.

Verified Answers to guide users toward predefined, validated responses.

And AI Instructions to provide additional business context.

Together, these capabilities help us make our semantic models more AI-ready.”

⸻

5. Validating the Copilot Experience

[Switch to the Copilot-enabled report or show a prepared test example.]

“Now, there’s one more thing I want to briefly touch on.

Even after completing all three steps, it’s important that we validate the Copilot experience before making it available to our users.

Because configuring the model is one part of the process, but making sure Copilot interprets our business questions correctly is equally important.

As developers, we should test the kinds of questions that our end users are likely to ask.

For example, we can ask Copilot a question about a business metric and compare its response with the results in our validated reports.

We want to make sure it’s using the right measures, understanding the right business definitions, and returning the expected results.

And if we identify any gaps, we can revisit the semantic model, improve our metadata, or refine our AI configuration.

So, the key point here is that AI readiness isn’t just about enabling a feature. It’s also about testing and continuously improving the experience.”

⸻

6. Closing — Final Takeaway

[Stop navigating and return your attention to the audience.]

“So, just to bring everything together.

We’ve covered three key areas developers should focus on when preparing their Power BI reports for Copilot.

First, following semantic modeling best practices and having a well-structured star schema.

Second, enriching our tables, columns, and measures with meaningful business metadata.

And third, using Prep data for AI to provide Copilot with additional context and guidance.

[Pause.]

Ultimately, the experience our end users have with Copilot depends heavily on how well we’ve prepared the data behind it.

The more effort we put into building clear, consistent, and well-governed semantic models, the better positioned we are to deliver reliable and meaningful Copilot experiences.

And that’s really the key takeaway: a good Copilot experience starts with a well-prepared semantic model.

With that, I’ll hand it back.

Thank you.”

[Hand over to the presenter or host.]