# ShopFlow Commerce — Business Analysis Practice Case

## 1. Project Overview

ShopFlow Commerce is a practice case for business analysis students. It is a mid-size online retailer selling products across four categories:

- Fashion
- Electronics
- Home
- Beauty

The company is not short of orders. It is losing potential customers before they buy, and it cannot clearly explain what happens after a purchase.

### The Problem in One Sentence

> Customers browse and leave, refunds are slow and invisible, and every team reports a different version of revenue.

Three facts from the synthetic dataset make this problem concrete:

- **Conversion funnel:** Only about 32% of sessions add something to the cart, and about 13% end in a purchase. The primary leak is before checkout, not at the payment button.
- **Refund visibility:** Returns exist, but customers cannot see whether their items have been received or when their money will be returned. Customer support fills this information gap with calls.
- **Inconsistent reporting:** Finance, merchandising, and marketing do not share a common definition of revenue, margin after refunds, or repeat purchase.

**The guiding principle:** Do not start with tools. Start with the business problem.

GitHub, Jira, wireframes, PostgreSQL, and Power BI are the tools used to demonstrate the solution. They are not the purpose of the project.

---

## 2. Project Objectives

The objective is to create a business analysis solution that helps ShopFlow Commerce:

1. Identify where potential customers leave the purchase funnel.
2. Improve the visibility and traceability of return and refund operations.
3. Establish consistent commercial definitions across business teams.
4. Reconcile reported figures against the underlying source data.
5. Translate business problems into testable requirements and user stories.
6. Deliver a reproducible analytical solution using PostgreSQL and Power BI.

### Expected Business Outcomes

- Better understanding of browse-to-buy conversion.
- Improved visibility into pending refunds and refund cycle times.
- Consistent reporting of revenue, orders, average order value, and refunded cash.
- Clearer understanding of category performance after refunds.
- More reliable customer cohort and RFM analysis.
- Traceability from business problems to requirements, implementation, and tests.

---

## 3. How to Approach the Project

Follow the steps in order. Skipping ahead is one of the most common reasons student projects become dashboards without a clear business decision behind them.

### Step 1: Define the Business Problem

Write the problem statement so that a project sponsor could agree or disagree with it.

Identify:

- **Who is affected?** Customers, customer support, finance, and merchandising.
- **What cannot they do today?** Track refunds, locate funnel drop-offs, and agree on commercial performance.
- **What will be measured?** Funnel conversion, refund processing status and cycle time, revenue, and contribution after refunds.

A useful problem statement must describe the business issue without relying on the word *dashboard*.

If the problem cannot be explained without mentioning a tool, the project is not ready for implementation.

### Step 2: Document the AS-IS and TO-BE Processes

#### AS-IS: Current State

- Customers email or contact support to ask about returns.
- The warehouse and finance teams do not share a common return ID.
- Customers cannot reliably see the status of a return or refund.
- Return processing cycle time is not consistently measured.
- Business teams calculate revenue and commercial metrics differently.

#### TO-BE: Future State

1. The customer opens an order and initiates a return.
2. The system generates a unique return ID.
3. The customer can track the return using a consistent status.
4. The warehouse and support teams see the same return status.
5. Refunds are processed only after the warehouse accepts the returned goods, according to the business rules.
6. The customer can see whether the return is being processed and whether the refund has been completed.

The five approved return statuses are:

| Status | Meaning |
|---|---|
| `Requested` | The return request has been created. |
| `In transit` | The returned item is on its way to the warehouse. |
| `Inspecting` | The warehouse is inspecting the returned item. |
| `Refunded` | The refund has been completed. |
| `Declined` | The return request has been declined. |

Use these same status names in requirements, wireframes, tests, and documentation.

### Step 3: Freeze the Scope

Define the scope before building the solution.

#### In Scope

- Refund status visibility and return tracking.
- Browse-to-buy funnel analysis.
- A common set of commercial metric definitions.
- Revenue and order reconciliation.
- Category contribution after refunds.
- Customer cohort and basic RFM analysis.
- Refund operations monitoring.

#### Out of Scope

- Developing a new mobile application.
- Rebuilding the loyalty programme.
- Building a tax calculation engine.
- Measuring advertising-spend ROI.
- Building a product recommendation engine.

**Scope discipline matters.** The existence of a column in the dataset, such as loyalty tier, does not automatically justify redesigning the loyalty programme.

Students should be able to explain why a feature is excluded, even when it seems interesting.

### Step 4: Convert Business Rules into Testable Requirements

Every business rule should become a requirement that can be tested.

Core business rules:

1. Cancelled orders do not count as revenue.
2. A return awaiting inspection is not a completed cash refund.
3. A refund cannot exceed the amount paid for the returned units.
4. Guest sessions must remain included in funnel analysis.
5. Customer cohort month is based on the first valid order, not the signup date.
6. Month zero of a customer cohort is 100% by definition and must not be presented as a retention achievement.
7. Revenue and refund calculations must use the same approved definitions across reports.

#### Example User Story

**Story ID:** `US-REF-001`

**Title:** Track the status of a return.

**User Story:**

As a customer, I want to see the status of my return so that I know whether my item has been received and when my refund has been completed.

**Acceptance Criteria — Given / When / Then**

- **Given** a customer has an eligible order, **when** they request a return, **then** the system creates a unique return ID and sets the status to `Requested`.
- **Given** a return has been shipped, **when** its status is updated, **then** the customer can see `In transit`.
- **Given** the warehouse receives the returned item, **when** inspection begins, **then** the status changes to `Inspecting`.
- **Given** the warehouse accepts the returned goods and the refund is completed, **when** the refund is recorded, **then** the status changes to `Refunded`.
- **Given** a return request is declined, **when** the decision is recorded, **then** the customer can see `Declined`.

A requirement is not complete until a tester can verify it and demonstrate a failure when the expected behavior is not met.

### Step 5: Model the Data at the Correct Grain

A fundamental data modeling principle is:

> One row should represent one clearly defined business entity or event.

Do not treat an order, order line, website session, and refund as interchangeable records.

| Data entity | Grain | Example use |
|---|---|---|
| Session | One website session | Funnel analysis |
| Order | One order | Order counts and order-level revenue |
| Order line | One purchased product line | Category contribution and unit-level returns |
| Return | One return request | Return status and processing time |
| Refund | One recorded refund transaction | Refunded cash |
| Customer | One customer | Cohort and RFM analysis |

#### Modeling Rules

- Do not count order lines as orders.
- Do not count sessions as purchases.
- Never add session-level revenue to order-level revenue.
- Prevent duplicated order totals when joining orders to multiple order lines.
- Calculate refund amounts using approved business rules.
- Use consistent definitions for revenue and contribution after refunds.

Implement the approved commercial calculations in **PostgreSQL views**, which form the analytical mart.

Power BI should read from the mart rather than independently redefining the same metrics.

This keeps the business logic consistent and makes the results easier to test.

### Step 6: Reconcile the Numbers Before Presenting Them

Load the synthetic CSV files into the database and validate the calculations before building visualizations.

For the basic order revenue reconciliation:

- Exclude cancelled orders.
- Sum `order_total` for orders whose status is not cancelled.
- Compare the SQL result with the executive revenue card.
- Investigate any difference before presenting the result.

Illustrative SQL:

```sql
SELECT
    SUM(order_total) AS revenue
FROM orders
WHERE status <> 'cancelled';
```

Adapt the table name, column names, and status values to the actual dataset and approved metric definition.

Additional validation should cover:

- Total order count.
- Average order value.
- Completed refunded cash.
- Funnel session counts.
- Funnel conversion percentages.
- Category contribution after refunds.
- Customer cohort sizes and retention rates.

**Do not publish a chart until its underlying numbers reconcile.**

Trust is the deliverable. Colour is not.

### Step 7: Design the Four Analytical Pages

Only after requirements, data modeling, and reconciliation are complete should students design the Power BI report.

#### Page 1: Executive Overview

Purpose: Give business leaders a consistent view of commercial performance.

Key metrics:

- Revenue.
- Orders.
- Average order value (AOV).
- Refunded cash.

Business questions:

- Is revenue changing?
- Is the change driven by order volume, basket size, or both?
- How much cash has actually been refunded?

#### Page 2: Merchandising

Purpose: Understand which categories perform well after accounting for returns and refunds.

Key metrics and analyses:

- Sales by category.
- Contribution after refunds.
- Return rates.
- Return reasons.
- Product and category performance.

Business questions:

- Which categories look healthy on sales but weak after refunds?
- Which return reasons occur most frequently?
- Where should merchandising investigate product descriptions, sizing, or quality?

#### Page 3: Customers

Purpose: Understand customer retention and purchasing behavior.

Key analyses:

- Customer acquisition cohorts.
- Cohort retention by month.
- Basic RFM segmentation.

RFM stands for:

- **Recency:** How recently a customer purchased.
- **Frequency:** How often a customer purchased.
- **Monetary:** How much a customer spent under the approved calculation rules.

Business questions:

- Are customers returning after their first purchase?
- Which customer segments purchase most frequently?
- How does retention change across acquisition cohorts?

Use the first valid order date to determine a customer's cohort month, rather than the signup date.

#### Page 4: Funnel and Refund Operations

Purpose: Identify where customers abandon purchases and where returns become operationally delayed.

Key analyses:

- Sessions by funnel stage.
- Add-to-cart conversion.
- Purchase conversion.
- Funnel comparison by device type.
- Pending refund age.
- Return status distribution.
- Refund processing cycle time.

Business questions:

- At which funnel stage are customers dropping off?
- Does the funnel differ between mobile and desktop?
- How many refunds are still pending?
- Which pending returns have been waiting the longest?

---

## 4. Technology Stack and Deliverables

Use tools to support the business analysis process rather than allowing the tools to define the project.

| Tool | Purpose | Expected deliverable |
|---|---|---|
| GitHub | Version control and project documentation | README, requirements, SQL, and datasets |
| Jira | Requirement and task management | Epics, user stories, acceptance criteria, and linked IDs |
| Wireframing tool | Interface and process design | AS-IS/TO-BE diagrams and screen wireframes |
| PostgreSQL | Data storage, transformations, and analytical logic | Database schema, SQL scripts, and analytical views |
| Power BI | Business reporting and visualization | Four-page analytical report |
| CSV files | Synthetic source data | Reproducible input datasets |

### GitHub Repository Structure

A suggested structure is:

```text
shopflow-commerce/
│
├── README.md
│
├── docs/
│   ├── business-problem.md
│   ├── scope-and-objectives.md
│   ├── as-is-to-be.md
│   ├── business-rules.md
│   ├── requirements.md
│   ├── data-dictionary.md
│   └── test-cases.md
│
├── data/
│   ├── customers.csv
│   ├── sessions.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── returns.csv
│   └── refunds.csv
│
├── sql/
│   ├── schema.sql
│   ├── load_data.sql
│   ├── transformations.sql
│   ├── analytical_views.sql
│   └── validation_queries.sql
│
├── wireframes/
│   ├── return-tracking.md
│   └── power-bi-layouts.md
│
├── power-bi/
│   └── README.md
│
└── tests/
    ├── business-rule-tests.md
    └── reconciliation-tests.sql
```

The filenames above are illustrative. Replace them with the actual dataset filenames and project artifacts.

### Jira Structure

Organize the work into epics, user stories, and testable acceptance criteria.

Suggested epics:

- `EPIC-01`: Commercial Metrics and Revenue Reconciliation.
- `EPIC-02`: Browse-to-Buy Funnel Analysis.
- `EPIC-03`: Return and Refund Visibility.
- `EPIC-04`: Customer Cohort and RFM Analysis.
- `EPIC-05`: Reporting and Validation.

Link each story to its requirement, relevant SQL or wireframe, and corresponding test case.

### Wireframe Requirements

Wireframes should cover more than the ideal success path.

Include:

- Initial and empty states.
- Return request creation.
- Return in transit.
- Warehouse inspection.
- Successful refund.
- Declined return.
- Validation errors.
- Unavailable or delayed status information.

Use the same five return status labels throughout the project.

---

## 5. What Students Should Say During a Review

Keep the review concise. Students should be able to explain the business findings in four sentences.

1. Revenue moved because of changes in order volume, basket size, or both.
2. One category looks healthy on sales but weak once refunds are deducted.
3. The funnel breaks at add-to-cart, and mobile and desktop may differ.
4. The next build is a specific story already in the backlog, not a new idea invented during the review.

These statements should be supported by the actual results from the synthetic dataset. Do not present them as proven findings until the analysis confirms them.

### If Asked: "Why Not Build a Recommendation Engine?"

The answer is that the largest identified funnel problem occurs before checkout, particularly at add-to-cart.

A recommendation engine does not automatically solve missing delivery information, unclear sizing guidance, or product descriptions that contribute to returns.

The team should investigate the actual causes of abandonment and returns before investing in additional personalization.

### If Asked: "What Would You Cut?"

Cut basket-saving before cutting refund status visibility.

Customers need to know where their returned items and money are. A saved basket is an optimization; an invisible refund is a trust problem.

---

## 6. What Good Looks Like

A successful submission should satisfy the following criteria.

### Reproducibility

- Another reviewer can load the supplied CSV files.
- The database schema can be recreated.
- SQL transformations and analytical views can be rerun.
- Revenue, funnel counts, and refunded-cash figures match the validated results.

### Business Consistency

- Revenue is calculated using one approved definition.
- Cancelled orders are excluded.
- Pending inspections are not treated as completed cash refunds.
- Refunds do not exceed the amount paid for the relevant units.
- Guest sessions are included in the funnel.
- Customer cohort month is based on the first valid order.

### Traceability

A reviewer can follow the same business rule through:

1. Business problem.
2. Requirement ID.
3. Jira user story.
4. Acceptance criteria.
5. SQL implementation.
6. Test case.
7. Power BI output, where applicable.

### Analytical Communication

Students can:

- Explain average order value without opening the source file.
- Explain the difference between revenue and refunded cash.
- Identify the main funnel drop-off.
- Explain why category sales alone may not indicate commercial health.
- Recommend a next step that is supported by the analysis and linked to an existing backlog story.

---

## 7. Important Note About the Data

The datasets, the 30-page dossier, and `schema.sql` form the worked example for this practice case.

The figures are **synthetic and intended for learning purposes only**. They must not be represented as actual ShopFlow Commerce trading results.

Students should explicitly disclose this limitation in their documentation and presentations.

The purpose is to demonstrate a reliable business analysis workflow, from problem definition and requirements to data modeling, validation, and reporting.

---

## 8. Final Principle

> Start with the business problem. Define the rules. Model the data correctly. Reconcile the numbers. Build the report only when the evidence is trustworthy.

The goal is not merely to produce a Power BI dashboard. It is to produce a business analysis solution that another person can reproduce, verify, explain, and use to make a decision.
