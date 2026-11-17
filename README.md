# Support Ticket Router - Homework 3

## Project Description

This project uses LangGraph and an OpenAI language model to classify customer support tickets for TaskNest, a to-do application.

The system classifies each ticket into one of three categories:

Billing: Payments, refunds, invoices, and subscriptions.

Technical: Bugs, crashes, login problems, and technical issues.

General: General questions, feature requests, and feedback.

## How It Works

The LangGraph workflow contains four nodes:

1. classify: Uses an LLM to understand the ticket and choose a category.
2. billing_team: Handles billing tickets.
3. tech_team: Handles technical tickets.
4. general_team: Handles general inquiries.

Conditional routing sends each ticket to exactly one support team. Team replies are generated using fixed Python text.

## Model

The project uses gpt-6-luna with structured output.

## Test Results

Nine support tickets were tested:

3 billing tickets

3 technical tickets

3 general tickets

All nine classifications matched the expected categories.

An ambiguous ticket about the app crashing during payment was correctly classified as a technical issue.

## How to Run

1. Open support_ticket_router.ipynb in Google Colab.
2. Add your OpenAI API key to Colab Secrets under the name OPENAI_API_KEY.
3. Enable Notebook access.
4. Run the notebook cells in order.

## Files

support_ticket_router.ipynb - Complete implementation, graph visualization, test results, and reflection.

## Security

The API key is loaded securely from Google Colab Secrets and is not stored in the notebook.
