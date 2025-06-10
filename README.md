# Query Builder Demo

A modern web application built with SvelteKit that demonstrates a powerful query builder interface with a performant data grid. The application allows users to create, manage, and execute complex queries with various condition types, and view the results in a virtualized data grid.

## Features

- **Query Builder**
  - Create and manage multiple queries
  - Support for different condition types:
    - Boolean (Yes/No)
    - Range (Min/Max values)
    - Text
    - Companies (Multiple selection)
  - Edit and remove conditions
  - View query history and results

- **Data Grid**
  - Virtual scrolling for optimal performance
  - Sortable columns
  - Pagination
  - Configurable page size
  - Loading states
  - Responsive design

## Tech Stack

- [SvelteKit](https://kit.svelte.dev/) - Full-stack web framework
- [TypeScript](https://www.typescriptlang.org/) - Type safety
- [TailwindCSS](https://tailwindcss.com/) - Utility-first CSS framework
- [UUID](https://github.com/uuidjs/uuid) - Unique ID generation
- [Faker](https://fakerjs.dev/) - Data generation for demos

## Project Structure

```
src/
├── lib/
│   └── components/
│       ├── DataGrid.svelte      # Virtualized data grid component
│       ├── QueryCollection.svelte # Query management component
│       └── QueryPills.svelte    # Query condition pills component
├── routes/
│   └── +page.svelte            # Main application page
└── app.html                    # HTML template
```

### Component Overview

- **DataGrid**: A performant data grid with virtual scrolling, sorting, and pagination
- **QueryCollection**: Manages the collection of queries and their execution
- **QueryPills**: Handles the creation and editing of query conditions

## Getting Started

### Prerequisites

- Node.js (v16 or later)
- pnpm (v7 or later)

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd ui-testing
   ```

2. Install dependencies:
   ```bash
   pnpm install
   ```

3. Start the development server:
   ```bash
   pnpm run dev
   ```

4. Open your browser and navigate to `http://localhost:5173`

### Development

The application uses SvelteKit's development server with hot module replacement. Any changes to the source files will automatically trigger a rebuild and refresh the browser.

### Building for Production

To create a production build:

```bash
pnpm run build
```

The built application will be in the `build` directory.

## Usage

1. **Creating a Query**
   - Click "New Query" to create a new query
   - Enter a name for your query
   - Add conditions using the "Add Condition" button
   - Select the condition type and configure its values

2. **Managing Queries**
   - View all your queries in the main interface
   - Set a query as current to edit it
   - Execute queries to see results
   - View historical queries and their results

3. **Viewing Results**
   - Click "Execute" on a query to view its results
   - Use the data grid to sort and paginate through results
   - Adjust the number of rows per page as needed

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
