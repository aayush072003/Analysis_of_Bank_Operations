Bank Data Analysis — Power BI Project

Repository: Bank-Data-Analysis

Purpose: Analyze the provided banking datasets to derive insights and validate a set of hypotheses about customer behaviour, transaction patterns, and branch performance — then present findings as an interactive Power BI report with visualizations, measures and clear answers to each hypothesis-driven question.

Table of contents

Project Summary

Datasets (expected files & sample schema)

Data preparation / Power Query steps

Data model / Relationships

Report pages & Visualizations (mapped to Hypotheses / Questions)

Recommended DAX measures & calculated columns (copyable examples)

Analysis approach & validation notes

Deliverables

Folder structure

Assumptions, thresholds & caveats

How to reproduce (quick start)

License & contact

1. Project Summary

You will build a Power BI report that validates the following hypotheses (Q1–Q8). For each hypothesis the README describes: the visual(s) to create, the DAX/measures required, the data prep steps, and how to interpret results so the visualization answers the question.

2. Datasets (expected files & sample schema)

Place all CSV/Excel files in the /data folder. Example files the analysis expects (rename or adapt to your provided files):

customers.csv — Customer-level master data

CustomerID (PK)

FullName

DateOfBirth (or Age)

Gender

ResidentialAreaType (Urban/Rural/Suburban) — if present

City, State

accounts.csv — Accounts master

AccountID (PK)

CustomerID (FK)

AccountType (Savings/Current/Salary/OverDraft etc.)

OpenDate

Balance

transactions.csv — Transaction ledger

TransactionID (PK)

AccountID (FK)

BranchID (FK)

TransactionDateTime (timestamp)

TransactionType (Deposit, Withdrawal, Transfer, Payment, etc.)

Amount

Currency (if multi-currency)

branches.csv — Branch master

BranchID (PK)

BranchName

AreaType (Urban / Rural / Semi-Urban)

City, State, Region

NumberOfEmployees

(Optional) employees.csv, cards.csv etc. — adapt as available

Important: If some columns are missing, adapt the steps below; we also include robust Power Query tips for cleaning and deriving missing fields (e.g., compute Age from DateOfBirth).

3. Data preparation / Power Query steps

Import: Load all CSVs/Excel sheets into Power BI Desktop (Get Data).

Type corrections: Ensure DateTime fields are Date/Time, amounts are Decimal, IDs are text.

Derive:

Age — if only DateOfBirth provided: Age = Date.Year(DateTime.LocalNow()) - Date.Year([DateOfBirth]) (or use exact days diff for accurate ages).

TransactionDate and TransactionTime — split TransactionDateTime to date and time.

DayOfMonth — Date.Day([TransactionDate]).

HourOfDay — Time.Hour([TransactionTime]).

Flag high-value transactions: Add column IsHighValue using a threshold (see Assumptions). Or compute percentile and set IsHighValue = Amount >= P95.

Account counts per customer: Group accounts by CustomerID and count account types; join back to customers to create AccountCount (1 vs. >1).

Trim & clean: Remove duplicated rows, handle nulls (e.g., missing branch ID or amount). Keep raw copies for audit.

Create Date dimension: Use Home > Transform data > New Table or generate in Power Query. This enables time intelligence.

4. Data model / Relationships

customers[CustomerID] 1-to-many → accounts[CustomerID]

accounts[AccountID] 1-to-many → transactions[AccountID]

branches[BranchID] 1-to-many → transactions[BranchID]

Date[Date] 1-to-many → transactions[TransactionDate]

Set Date as the primary date table and mark it as Date table in Power BI.

5. Report pages & Visualizations (mapped to Q1 — Q8)

For each hypothesis we recommend visuals, interactions, and how to answer the question.

Q1 — Account types by age groups

Hypothesis: Different age groups prefer different account types.

Visuals:

Stacked bar chart: X = AgeGroup (18-25, 26-35, 36-50, 51+), Y = Count of Accounts (or % of accounts), Legend = AccountType.

100% stacked bar alternative to show composition per age group.

Slicer: Gender, Region, Account open date range.

Interpretation guide: Look for dominant account types in each age group (e.g., Salary accounts concentrated in 18–35). Use percentages to control for group size.

Power Query prep: Create AgeGroup column.

Q2 — Urban vs Rural branch transaction amounts

Hypothesis: Urban branches have higher total transaction amounts.

Visuals:

Clustered column or donut: Branch AreaType (Urban / Rural / Semi-Urban) vs Sum(Transaction Amount).

Map visual (if branch coordinates exist): bubble size = total transaction amount.

Interpretation: Compare total sums and average transaction size per branch type.

Q3 — Transactions over a month (beginning & end peaks)

Hypothesis: Transactions spike at beginning and end of month.

Visuals:

Line chart: X = DayOfMonth (1–31), Y = Count(Transactions) aggregated across months (or per selected month). Optionally show moving average.

Heatmap: Day of Month vs HourOfDay to find intra-day patterns.

Interpretation: Look for local maxima around days 1–5 and 25–31.

Q4 — High-value transactions frequency by branch

Hypothesis: Certain branches have more high-value transactions.

Visuals:

Bar chart: BranchName vs Count(IsHighValue = TRUE).

Sorted descending; optionally show proportion of high-value to total transactions.

Interpretation: Identify top branches and check whether they correspond to urban/metropolitan areas.

Q5 — Customers with multiple account types vs single account average balance

Hypothesis: Multi-account customers have higher overall balances.

Visuals:

KPI cards: Avg Balance (Multi-account customers) vs Avg Balance (Single-account customers).

Boxplot or violin (if custom visual allowed) to compare distribution of balances.

How to compute:

Flag customers where AccountCount > 1.

Aggregate TotalBalancePerCustomer = SUM(accounts[Balance]) (or compute current total across accounts).

Compare averages and medians.

Q6 — Employees vs transaction volume relationship

Hypothesis: More employees -> larger transaction volume.

Visuals:

Scatter plot: X = NumberOfEmployees (branch), Y = TotalTransactionVolume (sum of Amount or count of transactions). Bubble size = Avg Transaction Amount; tooltips show branch name and region.

Add linear trendline to measure correlation.

Interpretation: Check correlation (positive, negative or none). Consider controlling for branch size and customer base.

Q7 — Transaction types by time of day

Hypothesis: Different transaction types peak at specific times.

Visuals:

Stacked area chart or clustered column: X = HourOfDay (0–23), Y = Count(transactions), Legend = TransactionType.

Heatmap (TransactionType × HourOfDay) showing counts.

Interpretation: e.g., Deposits peak in morning; withdrawals at lunch or evening.

Q8 — Transaction frequency by age groups

Hypothesis: Younger customers (18–35) transact more frequently.

Visuals:

Bar chart: AgeGroup vs Avg Transactions per Customer (or total transactions normalized by number of customers in the group).

Boxplot of transactions per customer by age group.

How to compute:

TransactionsPerCustomer = COUNT(transactions[TransactionID]) grouped by CustomerID; then average that per age group.

6. Recommended DAX measures & calculated columns

Below are copy-paste DAX snippets to accelerate building the report.

Date calculations

TransactionsCount = COUNT('transactions'[TransactionID])
TotalAmount = SUM('transactions'[Amount])

High value threshold (example absolute)

IsHighValue = IF('transactions'[Amount] >= 100000, 1, 0)
HighValueCount = CALCULATE(COUNTROWS('transactions'), FILTER('transactions', 'transactions'[Amount] >= 100000))

Alternative percentile threshold (top 5%): compute P95 amount in a measure and use it in filters.

Total balance per customer

TotalBalancePerCustomer = CALCULATE(SUM('accounts'[Balance]), ALLEXCEPT('accounts', 'accounts'[CustomerID]))

Account count per customer

AccountCount = CALCULATE(DISTINCTCOUNT('accounts'[AccountType]), ALLEXCEPT('accounts','accounts'[CustomerID]))
HasMultipleAccounts = IF([AccountCount] > 1, "Multiple", "Single")

Avg transactions per customer (age group)

TransactionsPerCustomer = COUNTROWS('transactions')
AvgTxPerCustomer = AVERAGEX(VALUES(customers[CustomerID]), CALCULATE(COUNTROWS('transactions')))

Branch transaction volume

Branch_TotalAmount = CALCULATE(SUM('transactions'[Amount]), ALLEXCEPT(branches, branches[BranchID]))
Branch_TransactionCount = CALCULATE(COUNTROWS('transactions'))

Employees vs volume correlation (Use scatter visual with a measure or column for NumberOfEmployees and Branch_TransactionCount.)

7. Analysis approach & validation notes

For each hypothesis:

Create the visual and relevant measures.

Test robustness: use absolute and relative metrics (counts and percentages).

Run sensitivity checks (e.g., change high-value threshold from absolute value to P90/P95 percentile and compare).

Summarize acceptance/rejection of hypothesis with evidence (charts + numeric summary).

Add tooltips to show exact numbers & filters.

Use bookmarks to create a guided story: Problem → Visualization → Conclusion → Suggested Actions.

8. Deliverables

PowerBI/BankDataReport.pbix — full report (pages matching Q1 to Q8 + Overview & Executive Summary)

/data — raw CSVs used

/scripts — any helper scripts used for preprocessing (optional)

Bank-Data-Analysis-README.md — this README

summary_insights.md — short bullet summary of conclusions for business stakeholders

9. Folder structure (suggested)
/Bank-Data-Analysis
  /data
    customers.csv
    accounts.csv
    transactions.csv
    branches.csv
  /PowerBI
    BankDataReport.pbix
  /scripts
    preprocess.ipynb (optional)
  Bank-Data-Analysis-README.md
  summary_insights.md
10. Assumptions, thresholds & caveats

Age groups: suggested bins — 18–25, 26–35, 36–50, 51+ (adjust if dataset shows different distributions)
