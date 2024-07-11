# Report Scheduler

## Update July 11 2024
Some notes as we revist a similar issue in another company.
- We ended up on this solution previously because we couldn't get recharts to look correct.
- We used https://github.com/yumauri/gotenberg-js-client on Next.JS since it renders server side.
- Finally, the easier solution is hosting server-side with https://react-pdf.org/ (no idea on library compatability).

## About
Report Scheduler creates PDF from React templates.

## Getting Started
1. Clone this repository.
2. Run `yarn` to fetch fresh packages.
3. Run `yarn dev` to start the Report Scheduler in development environment.
4. Navigate to your `localhost:3000` to view available templates.

Other running scripts can be observed in `package.json` i.e. `yarn test` for testing.

## Usage
Report Scheduler relies internally on [Gotenberg](https://github.com/gotenberg/gotenberg) to generate the actual reports.

> The Gotenberg version used currently is v6. v7 has a slightly different endpoint API. This constant, GOTENBERG_URL, is in `.env.development`.

For development, you may choose to run the Docker image provided by Gotenberg:

    docker run --rm -p 3333:3000 gotenberg/gotenberg:7

> We port forward the container to :3333 as :3000 is taken by Report Scheduler

Ensure your `localhost:3000` is live. The following endpoints are available.

1. `localhost:3000` - Glossary of all available templates.

2. `localhost:3000/templates/{template_name}` - Web view of a template.

3. `localhost:3000/api/SampleReport` - (GET) Sample PDF generation.

4. `localhost:3000/api/{template_name}` - (POST) Generates a PDF with the given template. An approriate JSON body must be fed through the POST call.

> In production, both the `gotenberg` and `report-scheduler` service can be found in the `common-svcs` namespace.

## Architecture
The report-scheduler service runs on Next.js. This is ideal as we like users working on the service to be able to:

1. Build or edit reports visually.
2. Be able to use React and Javascript libraries freely.

Because Next.js uses server side rendering, we are able to build reports using the same React framework and get the benefits of rendering to the DOM and handle API without wrangling with a regular React + Node.js solution.

![Architecture](/architecture.png)

## Structure
`/charts`

Contains landscape and horizontal bar charts for reports.

`/components`

Common components used in layouts.

`/constants`

Common constants used in layouts.

`/data`

Sample data used for reports.

`/layouts`

Layouts used for templates.

`/middleware`

Middleware functions for API validation.

`/pages/templates`

Templates for the reports.

`/pages/api`

API endpoints to access reports.

`/public`

Common public assests.

`/styles`

Common styles shared across reports.

`/types`

Compile time and run time types used to validate props and data.

`/utils`

Utility functions.

## Troubleshooting
This section discuss common errors faced when accessing the service's endpoints.

```
{
  "code": "Method Not Allowed",
  "description": "The HTTP method used for this API call is not allowed."
}
```

The `method` middleware layer is enabled on the endpoint and the HTTP method is not permitted.

```
{
  "code": "Bad Request",
  "description": "The request body for this API call is missing or malformed.",
  "error": [
    {
      "context": [
        {
          "key": "",
          "type": {
            "name": "{ reportTitle: string, frequency: number, startDate: string, endDate: string, threatsDetected: number, emailsImpacted: number, uniqueDomains: number, activeMailboxes: number, threatRemediation: Array<{ [K in string]: (string | number) }>, vipIncidents: number, vipIncidentsPercentage: number, vipIncidentsPoints: Array<number>, threatLandscape: Array<{ [K in string]: (string | number) }>, domains: (Array<{ [K in string]: (string | number) }> | undefined), spoofs: (Array<{ [K in string]: (string | number) }> | undefined), abuseIncidents: number, abuseThreatRemediation: Array<{ [K in string]: (string | number) }>, historicalThreatLandscape: Array<{ [K in string]: (string | number) }> }",
            "props": {
              "reportTitle": {
                "name": "string",
                "_tag": "StringType"
              },
              "frequency": {
                "name": "number",
                "_tag": "NumberType"
              },
              "startDate": {
                "name": "string",
                "_tag": "StringType"
              },

              ...

            ]
          }
        },
        {
          "key": "reportTitle",
          "type": {
            "name": "string",
            "_tag": "StringType"
          }
        }
      ]
    }
  ]
}
```

The `validator` middleware layer is enabled on the endpoint. The error is created by `io-ts` runtime validation and the JSON body does not match the expected format.

The JSON body is malformed and is missing a certain key or the value of a key is of the wrong type. You can observe the malignant property by going to the end of the JSON.

In the example above, reportTitle is `missing` or malformed. 
