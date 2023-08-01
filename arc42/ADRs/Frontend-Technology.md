# ADR: Use **React Native** as the frontend technology for PowerMate

## Status

Accepted

## Context

For powermate, a frontend solution is needed where users can see and understand their gas consumption. Two technologies where proposed by the team: React and React Native.

### React

**Pros:**

- Wider familiarity among team members
- Huge amount of libraries and community components
- Ability to leverage existing web development skills
- Easier deployment
- Easier styling

**Cons:**

- No mobile support
- Would need an extra app to integrate data collection

### React Native

**Pros:**

- Mobile support
- Ability to collect gas meter data
- Huge amount of libraries and community components
- Frameworks like Expo facilitate the distribution of the app with different channels, such as for instance *DEV*, *QA* and *PROD*

**Cons:**

- More difficult deployment and testing
- Steeper learning curve
- Dependency of Expo for build process

### Decision

Due to the importance of collecting data with a smartphone and the added value to the powermate-system, the team has decided to use React Native as the frontend technology.
