## Hotel Booking Performance Analytics Dashboard
## Power BI Business Intelligence Project
## Project Overview

This project develops an interactive Power BI business intelligence solution for hotel booking analysis. The solution transforms raw hotel reservation data into a structured analytical model that enables management to monitor booking demand, cancellation behaviour, customer characteristics, pricing, and estimated booking value.

The project was developed through four main stages:

Data cleaning and transformation using Power Query
Dimensional modelling using a star schema
Business calculations using DAX
Interactive dashboard development and business insight generation

The final report contains three analytical pages:

Executive Overview
Booking & Cancellation Analysis
Revenue & Pricing Analysis

The dashboards progress from high-level performance monitoring to detailed investigation of cancellation behaviour and finally deeper commercial analysis.

1. Business Problem

Hotel managers need to understand not only how many reservations are being made, but also the factors affecting booking performance.

A high number of reservations does not necessarily translate into successful hotel stays because bookings may be cancelled, customer behaviour differs between market segments, booking channels have different risk profiles, and room prices vary substantially.

The main business problem addressed by this project is therefore:

How can hotel booking data be transformed into actionable business intelligence that helps management monitor demand, understand cancellation behaviour, evaluate pricing and booking value, and identify opportunities to improve hotel performance?

The Power BI solution was designed to answer questions including:

How many bookings and guests are represented in the dataset?
What proportion of bookings are cancelled?
Which hotel type experiences greater cancellation risk?
Does cancellation behaviour change with booking lead time?
Which customer types have the highest cancellation rates?
Which distribution channels generate higher or lower cancellation risk?
What is the average daily rate (ADR)?
How long do guests typically stay?
Which market segments contribute most strongly to booking activity and estimated realized revenue?
How does estimated realized revenue change over time?
How does room pricing vary across reserved room types?

2. Dataset Overview

The dataset contains hotel booking records for two hotel categories:

City Hotel
Resort Hotel

The original dataset contained approximately:

119,390 booking records
More than 30 booking-related attributes

The variables cover several analytical areas including:

Booking information

Examples include:

Lead time
Cancellation status
Arrival date
Weekend nights
Weekday nights
Booking changes
Waiting-list days
Reservation status
Guest information

Examples include:

Adults
Children
Babies
Country
Customer type
Commercial information

Examples include:

Average Daily Rate (adr)
Market segment
Distribution channel
Deposit type
Reserved room type
Assigned room type
Agent information
Customer-service information

Examples include:

Required car-parking spaces
Special requests

Following data cleaning and validation, the final analytical fact table contained:

119,209 booking records

Each record in the final fact table represents one hotel booking.

3. Data Preparation and Power Query

A dedicated staging query named:

FactBooking_Staging

was used for data cleaning and transformation.

The staging query was subsequently referenced to create the final fact table and dimension tables. Load was disabled for the staging query after the analytical tables were created to prevent unnecessary duplication within the Power BI model.

3.1 Removal of unnecessary/sensitive fields

Fields that were not required for the analytical objectives or contained inappropriate/sensitive information were removed during preparation.

The objective was to retain only information that contributed to booking, customer, operational, cancellation, or commercial analysis.

3.2 Removal of company

The company field contained a very high proportion of missing values.

Because the field did not contain sufficient information to support reliable company-level analysis, it was removed rather than attempting to impute arbitrary company identifiers.

This reduced unnecessary sparsity in the analytical dataset.

3.3 Missing values in children

A small number of records contained missing values in the children field.

These values were replaced with:

0

This preserved otherwise usable booking records.

Because a missing value does not conclusively prove that a booking contained zero children, this treatment is recognised as an analytical assumption rather than an observed fact.

Applied step:

Replaced Null Children

3.4 Missing country information

The country field contained 488 blank/missing entries.

Rather than deleting these bookings or assigning them to an unsupported country, the blank values were standardised to:

Unknown

This retained the booking information while transparently identifying unavailable geographic information.

Applied step:

Replaced Blank Country

3.5 Agent booking classification

The agent field contained a substantial number of missing values.

Because an agent number is an identifier rather than a numerical measurement, replacing missing IDs with an average, median, or arbitrary agent number would be inappropriate.

Instead, a new analytical category was created:

Agent_Booking_Status

Bookings were classified as:

Agent Booking
No Recorded Agent

Applied step:

Created Agent Booking Status

This preserves the original agent information while providing a more useful field for categorical analysis.

4. Derived Fields

Several additional fields were created to improve analytical usefulness.

4.1 Total Guests

Guest information was originally distributed across:

Adults
Children
Babies

A new field was created:

Total_Guests = adults + children + babies

Applied step:

Created Total Guests

This provides a single measure of booking party size and enabled additional data-quality validation.

4.2 Invalid zero-guest bookings

Data profiling identified:

180 records where Total_Guests = 0

These records contained:

0 adults
0 children
0 babies

Such records were considered inconsistent with the analytical definition of a usable guest booking.

The records were therefore excluded.

Applied step:

Removed Zero Guest Bookings

This reduced the dataset from:

119,390 → 119,210 records

Only approximately 0.15% of the original dataset was affected.

5. ADR Validation

adr represents Average Daily Rate and is an important commercial variable.

Data profiling identified:

Minimum ADR: -6.38
Maximum ADR: 5,400
Average ADR before final filtering: approximately 101.97
Zero ADR observations were also present.

A negative room rate was considered inconsistent with the intended interpretation of ADR because the dataset did not provide supporting information identifying the value as a refund or accounting adjustment.

Therefore:

Records where ADR < 0 were excluded.

Applied step:

Removed Negative ADR

Only one record was removed.

The final dataset therefore contained:

119,209 records

Zero ADR records were retained because zero values may represent legitimate booking circumstances.

The maximum ADR of 5,400 was also retained. Although extreme, an outlier is not automatically an error, and insufficient evidence existed to justify removing the observation.

6. Data-Type Correction

Automatically detected Power BI data types were reviewed and corrected according to the business meaning of each field.

Examples include:

Guest counts → Whole Number
Agent identifier → Whole Number
ADR → Decimal Number
Reservation dates → Date
Booking-status fields → Text
Total Guests → Whole Number

Applied step:

Corrected Data Types

Correct data types were necessary to support accurate aggregation, filtering, relationships, date analysis and DAX calculations.

7. Arrival Date Construction

Arrival information originally existed as separate components including:

Arrival year
Arrival month
Arrival day

A complete date field named:

Arrival_Date

was constructed in Power Query.

This enabled proper chronological analysis and a relationship with the dedicated Date dimension.

Applied steps included:

Created Arrival Date
Corrected Arrival Date Data Type
8. Length of Stay

The dataset separately recorded:

Weekend nights
Weekday nights

A new field was created:

Length_of_Stay =
stays_in_weekend_nights + stays_in_week_nights

This provides the total planned duration associated with each booking.

Applied steps:

Created Length of Stay
Corrected Length of Stay Type
8.1 Zero-night investigation

Data profiling identified:

645 bookings with Length_of_Stay = 0

These records were temporarily isolated and their reservation statuses investigated.

Zero-night records occurred across:

Check-Out
Canceled
No-Show

Importantly, Check-Out represented the majority.

There was therefore insufficient evidence to classify zero-night bookings as erroneous.

The temporary investigation filter was removed and the 645 records were retained.

This decision demonstrates that unusual observations were investigated rather than automatically deleted.

9. Final Fact Table

A reference of the cleaned staging query was created and named:

FactBookings

The grain of this table is:

One row = one hotel booking

Redundant arrival-date fragments were removed after the complete Arrival Date field had been created.

A unique index was then generated:

Booking_ID

The final table contains:

119,209 unique Booking_ID values

This identifier provides a reliable basis for booking-count measures.

10. Dimensional Model

A star-schema approach was implemented.

FactBookings forms the centre of the analytical model and contains booking-level measures and transactional attributes.

Seven dimension tables were created.

DimDate

Contains a dedicated calendar structure including:

Date
Year
Quarter
Month Number
Month Name
Week Number
Day
Day Name

The table contains 793 unique dates.

DimHotel

Contains:

City Hotel
Resort Hotel
DimMarketSegment

Contains 8 unique market segments.

DimDistributionChannel

Contains 5 unique distribution channels.

DimCustomerType

Contains four customer types:

Transient
Contract
Transient-Party
Group
DimCountry

Contains 178 unique country categories, including Unknown.

DimRoomType

Contains 9 unique reserved-room-type categories.

11. Relationships

Each dimension table is connected to FactBookings using a:

One-to-Many (1:*) relationship

The dimension resides on the 1 side, while FactBookings resides on the many (*) side.

Single-direction filtering is used from dimensions toward the fact table.

Conceptually:

```md
                         DimDate
                            |
                            |
DimHotel ----------- FactBookings ----------- DimCountry
                         /   |   \
                        /    |    \
              DimRoomType    |    DimCustomerType
                             |
                     DimMarketSegment
                             |
                  DimDistributionChannel

```

This design:

reduces relationship ambiguity;
supports predictable filter propagation;
avoids unnecessary many-to-many relationships;
avoids unnecessary bidirectional filtering;
provides a scalable analytical structure.
12. DAX Business Measures

The project contains more than the minimum required number of analytical measures.

Major measures include:

Booking KPIs
Total Bookings =
DISTINCTCOUNT(FactBookings[Booking_ID])
Cancelled Bookings =
CALCULATE(
    [Total Bookings],
    FactBookings[is_canceled] = 1
)
Non-Cancelled Bookings =
CALCULATE(
    [Total Bookings],
    FactBookings[is_canceled] = 0
)
Cancellation Rate % =
DIVIDE(
    [Cancelled Bookings],
    [Total Bookings],
    0
)
Guest and operational KPIs
Total Guests =
SUM(FactBookings[Total_Guests])
Average Guests per Booking =
DIVIDE(
    [Total Guests],
    [Total Bookings],
    0
)
Average Length of Stay =
AVERAGE(FactBookings[Length_of_Stay])
Average Lead Time =
AVERAGE(FactBookings[lead_time])
Average Special Requests =
AVERAGE(FactBookings[total_of_special_requests])
Average Booking Changes =
AVERAGE(FactBookings[booking_changes])
Average Waiting List Days =
AVERAGE(FactBookings[days_in_waiting_list])
Pricing KPI
Average ADR =
AVERAGE(FactBookings[adr])
13. Advanced DAX

Advanced calculations were incorporated to demonstrate filter-context manipulation, ranking and time intelligence.

Previous Year Bookings
Previous Year Bookings =
CALCULATE(
    [Total Bookings],
    SAMEPERIODLASTYEAR(DimDate[Date])
)
YoY Booking Growth %
YoY Booking Growth % =
DIVIDE(
    [Total Bookings] - [Previous Year Bookings],
    [Previous Year Bookings],
    0
)

Because the dataset does not contain complete coverage for every calendar year, year-over-year results must be interpreted within the date context being compared.

Market Segment Booking Share %
Market Segment Booking Share % =
DIVIDE(
    [Total Bookings],
    CALCULATE(
        [Total Bookings],
        ALL(DimMarketSegment[market_segment])
    ),
    0
)

This measure demonstrates filter-context manipulation by comparing an individual segment's bookings against total bookings across all market segments.

Market Segment Rank
Market Segment Rank =
RANKX(
    ALL(DimMarketSegment[market_segment]),
    [Total Bookings],
    ,
    DESC,
    DENSE
)

This dynamically ranks market segments according to booking volume.

14. Revenue and Booking-Value Measures

Commercial measures were created using ADR and length of stay.

Estimated Booking Value
Estimated Booking Value =
SUMX(
    FactBookings,
    FactBookings[adr] * FactBookings[Length_of_Stay]
)

Result:

42.71M

This represents the estimated accommodation value associated with all booking records.

Estimated Realized Revenue
Estimated Realized Revenue =
SUMX(
    FILTER(
        FactBookings,
        FactBookings[is_canceled] = 0
    ),
    FactBookings[adr] * FactBookings[Length_of_Stay]
)

Result:

25.99M

Estimated Cancelled Booking Value
Estimated Cancelled Booking Value =
[Estimated Booking Value] -
[Estimated Realized Revenue]

Result:

Approximately 16.73M

Realized Revenue Rate %
Realized Revenue Rate % =
DIVIDE(
    [Estimated Realized Revenue],
    [Estimated Booking Value],
    0
)

Result:

60.8%

These measures are analytical estimates rather than accounting measures. The dataset does not provide evidence that cancelled inventory was never subsequently resold.

For this reason, the project deliberately uses the terminology:

Estimated Cancelled Booking Value

rather than claiming the amount represents proven lost revenue.

15. Analytical Grouping

Additional calculated columns were created to improve dashboard readability.

Cancellation Status

The original is_canceled values of 0 and 1 were converted into readable categories:

Cancellation Status =
IF(
    FactBookings[is_canceled] = 1,
    "Cancelled",
    "Not Cancelled"
)
Lead-Time Bands

Rather than presenting hundreds of individual lead-time values, bookings were grouped into:

0–7 Days
8–30 Days
31–90 Days
91–180 Days
181–365 Days
366+ Days

A separate numerical sort column ensures chronological ordering.

Length-of-Stay Bands

Length of stay was grouped into:

0 Nights
1–2 Nights
3–4 Nights
5–7 Nights
8–14 Nights
15+ Nights

This significantly improves the readability of stay-duration analysis.

16. Dashboard Design

The report contains three dashboard pages designed to progress from overview to deeper analysis.

Page 1 — Executive Overview
Purpose

Provides management with an immediate overview of hotel booking performance.

KPI Cards

The page presents:

Total Bookings
Cancellation Rate
Average ADR
Average Lead Time
Average Length of Stay

Key overall results include:

KPI	Result
Total Bookings	~119K
Cancellation Rate	37.08%
Average ADR	101.97
Average Lead Time	104.1 days
Average Length of Stay	3.43 nights
Total Guests	~235K
Average Guests per Booking	1.97
Visual Analysis

The page includes analysis of:

Booking trends over time
Bookings by hotel type
Bookings by market segment
Slicers

Interactive slicers include:

Hotel Type
Year

These enable users to dynamically filter the executive KPIs and supporting charts.

Page 2 — Booking & Cancellation Analysis
Purpose

Investigates the booking characteristics associated with cancellation behaviour.

Visuals include:

Booking Cancellation Status
Cancellation Rate by Hotel Type
Cancellation Rate by Lead Time
Cancellation Rate by Customer Type
Cancellation Rate by Distribution Channel
Average ADR by Cancellation Status
Bookings by Length of Stay

This page shifts from simply describing performance to investigating potential cancellation-risk characteristics.

Page 3 — Revenue & Pricing Analysis
Purpose

Examines the commercial value associated with bookings and non-cancelled reservations.

KPI Cards
Estimated Booking Value — 42.71M
Estimated Realized Revenue — 25.99M
Estimated Cancelled Booking Value — 16.73M
Average ADR — 101.97
Realized Revenue Rate — 60.8%
Supporting Visuals

The page analyses:

Estimated realized revenue by hotel type
Estimated realized revenue over time
Average ADR by reserved room type
Estimated realized revenue by market segment
Estimated realized revenue by distribution channel
17. Interactivity

The dashboard was designed as an interactive analytical report rather than a collection of static charts.

Interactivity includes:

Hotel-type slicers
Year filtering
Cross-filtering between visuals
Dimension-based filtering
Interactive selection of market segments
Interactive hotel comparisons
Dynamic recalculation of DAX measures according to filter context

For example, selecting City Hotel updates the KPI cards and supporting visuals to show only City Hotel performance.

Selecting a market segment similarly filters other visuals and recalculates relevant measures.

18. Key Findings
18.1 Overall cancellation rate is substantial

Approximately:

37.08% of bookings were cancelled.

This means cancellation management represents a significant operational and commercial issue.

18.2 City Hotel experiences substantially greater cancellation risk

Cancellation rates were:

City Hotel — approximately 41.8%
Resort Hotel — approximately 27.8%

City Hotel therefore experiences substantially greater cancellation exposure than Resort Hotel.

Management should investigate differences in customer mix, distribution channels, lead times and booking conditions between the two hotel types.

18.3 Cancellation risk rises dramatically with lead time

Cancellation rates by lead-time band were approximately:

Lead Time	Cancellation Rate
0–7 Days	9.6%
8–30 Days	27.9%
31–90 Days	37.7%
91–180 Days	44.7%
181–365 Days	55.5%
366+ Days	67.7%

This is one of the strongest findings in the analysis.

Bookings made more than one year in advance have a cancellation rate several times higher than bookings made within one week of arrival.

The result demonstrates a strong association between longer lead times and cancellation risk.

It does not establish that longer lead time directly causes cancellation.

19. Customer-Type Findings

Cancellation rates differ considerably across customer types.

Approximately:

Transient — 40.3%
Contract — 31.0%
Transient-Party — 25.5%
Group — 10.1%

Transient customers therefore represent the highest cancellation-risk customer category in the analysis, while Group bookings show considerably greater booking stability.

20. Distribution-Channel Findings

Cancellation behaviour also varies substantially by distribution channel.

Approximately:

Distribution Channel	Cancellation Rate
Undefined	80.0%
TA/TO	41.1%
Corporate	22.1%
GDS	19.2%
Direct	17.5%

Among identified channels, TA/TO bookings exhibit the highest cancellation rate, while Direct bookings exhibit the lowest.

The Undefined category should be interpreted cautiously because the distribution channel was not properly classified.

Its unusually high cancellation rate may also represent a data-quality or business-classification issue worthy of investigation.

21. ADR and Cancellation

Average ADR differs modestly according to cancellation outcome:

Cancelled — 105.02
Not Cancelled — 100.17

Cancelled bookings therefore have a slightly higher average ADR.

This suggests an association between higher-priced bookings and cancellation behaviour; however, the difference alone does not establish that higher ADR causes cancellation.

Other variables such as lead time, market segment, hotel type and distribution channel may also influence the relationship.

22. Length-of-Stay Findings

After grouping booking duration into practical categories, booking volumes were approximately:

Length of Stay	Bookings
0 Nights	0.65K
1–2 Nights	48.64K
3–4 Nights	44.44K
5–7 Nights	20.27K
8–14 Nights	4.80K
15+ Nights	0.43K

The majority of reservations therefore involve relatively short stays, particularly 1–4 nights.

The overall average stay is:

3.43 nights

23. Commercial Findings

The analysis estimates:

42.71M in total booking value
25.99M associated with non-cancelled bookings
16.73M associated with cancelled bookings
60.8% realized-value rate

These results highlight the potential commercial significance of cancellation management.

However, the 16.73M figure should not be interpreted as confirmed lost revenue because rooms from cancelled bookings may have subsequently been resold.

24. Business Recommendations

Based on the dashboard findings, several actions are recommended.

1. Introduce lead-time-based cancellation management

Long-lead bookings should receive greater attention because cancellation rates increase substantially with advance booking time.

Management could consider:

automated booking reconfirmation;
reminder communications;
staged deposits;
carefully designed cancellation conditions;
targeted pre-arrival engagement.

Particular attention should be given to bookings made more than 180 days in advance.

2. Investigate City Hotel cancellation drivers

City Hotel's cancellation rate is considerably higher than Resort Hotel's.

Management should investigate whether this is associated with:

customer mix;
booking channel;
lead time;
market segment;
pricing;
deposit conditions.
3. Review TA/TO booking strategy

TA/TO bookings show considerably higher cancellation rates than Direct bookings.

Management should compare the benefits of TA/TO booking volume against cancellation exposure and associated commercial terms.

This does not mean TA/TO channels should simply be reduced; their booking volume and realized value should also be considered.

4. Encourage direct-booking relationships

Direct bookings show a relatively low cancellation rate.

Where commercially appropriate, hotels could encourage direct booking through:

loyalty incentives;
direct-booking benefits;
personalised communication;
competitive direct rates;
simplified booking experiences.
5. Investigate Undefined distribution-channel records

The Undefined category has an unusually high cancellation rate.

Management should determine whether this reflects:

data-entry problems;
system integration problems;
unclassified booking sources;
or a genuine booking channel.

Improving source classification would strengthen future analysis.

6. Focus retention activity on transient customers

Transient bookings show the highest cancellation rate among the identified customer types.

Targeted communication and confirmation strategies may help reduce uncertainty in this segment.

7. Use estimated cancelled booking value as a risk indicator

The estimated value associated with cancelled bookings is commercially significant.

Management should use this as an exposure indicator, while recognising that it is not equivalent to confirmed accounting loss.

25. Assumptions

Several assumptions were necessary during development.

Missing children

Missing children values were replaced with zero to preserve a very small number of otherwise usable records.

This is an imputation assumption and does not prove that the original bookings contained no children.

Unknown countries

Missing country information was categorised as Unknown rather than assigning an unsupported country.

Agent information

Missing agent identifiers were not statistically imputed because agent numbers represent identifiers rather than continuous numerical measurements.

Zero-night bookings

Zero-night bookings were retained after investigation showed that many were associated with Check-Out reservation status.

Negative ADR

The single negative ADR record was excluded because no supporting information explained the negative room-rate value.

High ADR values

Extreme positive ADR observations were retained because unusual values were not automatically assumed to be errors.

Revenue estimation

Estimated booking value was calculated as:

ADR × Length of Stay

Estimated realized revenue includes only bookings classified as non-cancelled.

These calculations are analytical estimates rather than audited hotel revenue.

26. Limitations

The analysis has several limitations.

Revenue is estimated

The dataset does not contain a complete accounting ledger.

Therefore, estimated realized revenue should not be interpreted as audited financial revenue.

Cancelled rooms may have been resold

A cancelled booking does not necessarily represent permanently lost revenue.

The same room may subsequently have been sold to another customer.

Causality cannot be established

Relationships such as:

longer lead time → higher cancellation rate

represent associations.

The dashboard alone cannot establish causal relationships.

Partial-year comparisons

The available date range does not provide equally complete coverage for every calendar year.

Therefore, year-over-year measures should be interpreted within appropriate comparable date periods rather than treating every annual total as directly equivalent.

Unknown/Undefined categories

Some source information is unavailable and is represented through categories such as:

Unknown
Undefined

These categories should be interpreted separately from known business categories.

Room-type codes

Room types are represented using coded categories such as A, B, C, etc.

Because the dataset does not provide sufficient descriptive room names, the dashboard retains the original room codes rather than inventing unsupported descriptions.

27. Tools and Technologies

The project was developed using:

Microsoft Power BI Desktop
Power Query / M
DAX
Dimensional modelling / Star Schema
GitHub for project documentation and submission
28. Project Structure

A recommended repository structure is:

Hotel-Booking-PowerBI/
│
├── README.md
│
├── Hotel_Booking_Analytics.pbix
│
├── data/
│   └── hotel_bookings.csv
│
└── screenshots/
    ├── power_query_transformations.png
    ├── star_schema_model.png
    ├── executive_overview.png
    ├── booking_cancellation_analysis.png
    ├── revenue_pricing_analysis.png
    └── dax_measures.png

Note: The actual dataset filename and repository filenames should match the files used in the final submission.

29. Dashboard Screenshots
# Executive Overview

![Executive Overview](https://github.com/mwmichelle/Hotel-Bookings-DSA3050-EndSem/blob/main/dashboard%20screenshots/executive%20overview%20pg1.png)

The Executive Overview provides senior management with high-level KPIs, booking trends, hotel comparisons and market-segment performance.

# Booking & Cancellation Analysis

![Booking Analysis](https://github.com/mwmichelle/Hotel-Bookings-DSA3050-EndSem/blob/main/dashboard%20screenshots/booking%20analysis%20pg%202.png)

This page investigates cancellation behaviour across lead time, hotel type, customer type, distribution channel, ADR and length of stay.

# Revenue & Pricing Analysis

![Revenue Analysis](https://github.com/mwmichelle/Hotel-Bookings-DSA3050-EndSem/blob/main/dashboard%20screenshots/revenue%20and%20pricing%20pg%203.png)


This page analyses estimated booking value, estimated realized revenue, pricing, market segments, distribution channels and room types.

# Data Model

![Model View](https://github.com/mwmichelle/Hotel-Bookings-DSA3050-EndSem/blob/main/model%20view/model%20view.png)

The model follows a star-schema design centred around FactBookings, with seven analytical dimensions and one-to-many, single-direction relationships.

Power Query Transformation Evidence

power query/staging table applied steps.png


The staging layer documents the cleaning, missing-value handling, validation, derived fields and data-type corrections applied before modelling.

30. Conclusion

This project demonstrates the complete development of a Power BI business intelligence solution from raw hotel booking data through to management-level decision support.

The final solution integrates:

structured data cleaning;
evidence-based treatment of missing and anomalous values;
Power Query transformations;
a star-schema data model;
a dedicated Date dimension;
one-to-many relationships;
DAX business measures;
advanced filter-context calculations;
time intelligence;
interactive dashboards;
and business-focused recommendations.

The analysis identifies cancellation management as a major opportunity, with an overall cancellation rate of approximately 37.08%. Cancellation behaviour varies substantially across hotel types, customer types, distribution channels and, most notably, booking lead time. At the same time, the commercial dashboard demonstrates the importance of considering booking value alongside booking volume.
