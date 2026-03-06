# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a specification repository for a "Global Multiplication Trainer" — a client-side web application designed to bring children to automatic mastery of multiplication tables (≤2 seconds per problem, ≥95% accuracy) using adaptive algorithms and dopamine-driven reward mechanics inspired by TikTok.

**Status**: Specification phase. No code implementation yet.

## Key Specifications

The project is defined by three core documents:

- **[DescriptionOfMechanics.md](DescriptionOfMechanics.md)** — Product philosophy, behavioral mechanics, and UX principles. Read this to understand the "why" behind the design.
- **[KernelSpecification.md](KernelSpecification.md)** — Technical specification including data structures, exact formulas for the adaptive algorithm, and reward economics. This is the source of truth for all numerical constants and calculations.
- **[SystemPrompt.md](SystemPrompt.md)** — Development guidelines for implementing the application as a single HTML file with embedded CSS/JS.

## Core Architecture

The application follows a strict modular architecture:

1. **Engine** — Core game loop and state management
2. **Adaptive Algorithm** — Mastery score calculation and personalized problem selection
3. **Reward System** — XP economy, streaks, variable reinforcement events
4. **State Manager** — Centralized appState with localStorage persistence
5. **UI Renderer** — Vertical, TikTok-style interface

## Technical Constraints

- Single HTML file with embedded CSS and JavaScript
- No server-side components
- No external libraries (ES6+ native JavaScript only)
- Client-side only (localStorage for persistence)
- Micro-sessions of 60-90 seconds

## Key Formulas (from KernelSpecification.md)

**EMA for reaction time**: `EMA_new = 0.25 * current + 0.75 * EMA_old`

**Mastery Score**: Combines accuracy (60%), speedFactor (30%), and stabilityIndex (10%), with forgetting decay applied

**Priority for problem selection**: `(100 - masteryScore) * 0.6 + decay * 100 * 0.3 + randomBoost`

**Automation threshold**: attempts ≥ 100, accuracy ≥ 0.95, avgReactionMs ≤ 2000, masteryScore ≥ 85

## Development Philosophy

This is **not a game** — it's a behavioral training system based on:
- Stimulus → Response → Instant Reinforcement → Progress
- Variable reward schedule (5% x2, 3% blitz, 1% boss events)
- Spaced repetition with decay for forgotten items
- Fatigue tracking to adjust engagement

When implementing, prioritize speed, dynamics, unpredictability, and the feeling of movement over visual richness.
