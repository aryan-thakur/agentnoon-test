# Agentnoon Coding Test - Organizational Chart Visualizer

## Overview

This application visualizes an organizational hierarchy from a CSV file (`gigacorp.csv`). It displays employee details (Name, Job Title, Department, Location, Level, Photo) and calculates various cost metrics (Manager Cost, Individual Contributor Cost, Total Cost, and Management Cost Ratio). The visualization is interactive, allowing users to expand and collapse parts of the tree.

## Features

*   Visualizes organizational structure from `gigacorp.csv`.
*   Calculates and displays Manager Cost, IC Cost, Total Cost, and Management Cost Ratio.
*   Interactive node expansion and collapse.
*   Clear visual lines connecting managers to their direct reports.
*   Displays key employee attributes: Name, Job Title, Department, Location, Level, and a placeholder photo.

## Technologies Used

*   `Vue 3`
*   `Vite`
*   `Tailwind CSS`
*   `D3.js` (for data processing)
*   `LeaderLine` (for drawing connector lines)

## Setup and Running Instructions

### Prerequisites

*   Node.js (v18.x or later recommended)
*   npm (comes with Node.js)

### Steps

1.  Clone the repository: `git clone https://your-repository-url/agentnoon-test.git` (Replace with the actual URL)
2.  Navigate to the project directory: `cd agentnoon-test`
3.  Install dependencies: `npm install`
4.  Start the development server: `npm run dev`
5.  Open your browser and go to the URL provided by `Vite` (typically `http://localhost:5173`).

## Project Structure

*   `public/gigacorp.csv`: The sample data file for the organization.
*   `src/App.vue`: Main application component; handles data fetching, processing, and overall layout.
*   `src/components/TreeNode.vue`: Component responsible for rendering individual employee nodes and their children.
*   `src/main.js`: Entry point for the `Vue` application.
*   `vite.config.js`: `Vite` configuration file.
*   `tailwind.config.js`: `Tailwind CSS` configuration.

## How it Works

*   The application loads employee data from `public/gigacorp.csv`.
*   `src/App.vue` uses `D3.js` (`d3.stratify`) to convert the flat CSV data into a hierarchical structure.
*   It then recursively calculates cost metrics for each node (Manager Cost, IC Cost, Total Cost, and Management Cost Ratio).
*   The `src/components/TreeNode.vue` component is used recursively to display each employee as a card.
*   Users can click on a manager's card to expand or collapse the view of their direct reports.
*   `LeaderLine` dynamically draws SVG lines to connect managers to their reports when a node is expanded.

## Notes on Data (`gigacorp.csv`)

*   The CSV file must contain at least "Employee Id" and "Manager" columns for the hierarchy to be built.
*   "Name", "Job Title", "Salary", "Department", "Location", "Photo" (URL), and "level" are used to display information on employee cards.
*   Salary data is used for cost calculations. Non-managerial roles are considered Individual Contributors (ICs).

## Possible Future Enhancements

*   Allow editing of employee data directly within the application.
*   Support for alternative data sources (e.g., JSON files, APIs).
*   Implement search and filtering capabilities for employees.
*   Add more sophisticated visual customization options (e.g., themes, layouts).
*   Performance optimizations for very large datasets.
