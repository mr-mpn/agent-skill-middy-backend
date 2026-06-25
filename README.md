# AI Skill: Middy Serverless

An installable AI Agent skill that teaches any AI coding assistant how to properly architect and build Node.js serverless backends using the Middy middleware framework.

## What is this?

When asking an AI agent to build a serverless backend, they often default to outdated patterns, scattered Lambda functions, or raw AWS SDK code. 

This repository contains a `SKILL.md` file designed specifically for **AI consumption**. By providing this skill to your agent, you instruct it to use a modern, single-entry-point architecture leveraging `@middy/core` and `@middy/http-router`.

## Features Taught to the Agent

- **Single Entry Point**: Routing all traffic through one Lambda function (`/{proxy+}`) for faster local development and simpler infrastructure setup.
- **Middy Routing**: Using `@middy/http-router` to handle REST endpoints cleanly.
- **Middleware Chains**: Standardized request parsing, centralized error handling, and JWT authentication middleware.
- **Step-by-Step Scaffolding**: Instructs the agent exactly which npm packages to install (`@middy/core`, `serverless-offline`, etc.) and how to configure ES Modules in `package.json`.

## How to Use

1. Obtain the `skills/SKILL.md` file from this repository.
2. Provide it to your AI coding assistant. Depending on your tool, you can:
   - Add it as a global "rule" or "custom instruction".
   - Install it as a plugin/skill in supported agentic IDEs.
   - Simply attach the file to your chat and say: *"Read this skill first."*
3. Prompt the agent: *"Create a new serverless backend for my app using the Middy Serverless skill."*
4. The agent will read the instructions and scaffold a perfectly structured API from scratch!

## Repository Structure

- `skills/SKILL.md`: The core markdown file containing the architectural blueprint and instructions for the agent.
- `plugin.json`: Metadata for IDEs that support direct skill/plugin installation.
