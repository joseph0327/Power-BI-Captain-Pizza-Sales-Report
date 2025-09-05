
# Captain Pizza’s – Weekly Platform Payouts Dashboard (Power BI)

![Dashboard Overview](https://github.com/joseph0327/Power-BI-Captain-Pizza-Sales-Report/blob/b54b5cad5fe8d25b2ff438aab1d0e07d55709e6b/visual%201.png)

## 📊 Business Context  
This dashboard gives management a weekly view of platform payouts for **Captain Pizza’s** stores across different delivery platforms (Uber Eats, DoorDash, Menulog).  
It shows:  
- How much revenue is generated per week.  
- How much is deducted for commission and advertising.  
- What the actual payouts are.  

Essentially, it’s a **profitability view** at store & platform level.

---

## 🖥️ Dashboard Walkthrough  

### **Header KPIs**
- **Net Sales** – Total sales processed through each platform before deductions.  
- **Net Payout** – Amount received after commission & advertising.  
- **Commission Fee** – Absolute value of platform commissions.  
- **Commission Fee % of Sales** – Proportion of sales taken as commission.  
- **Advertising % of Sales** – Proportion spent on advertising / in-app promos.  
- **Advertising Spend** – Absolute advertising spend value.  
- **Take Rate %** – Total percentage lost to commission + advertising.

### **Detailed Table**  
- Weekly breakdown by **Store Name**, **Platform** and **Week Ending Sunday**.  
- Columns: Net Sales, Net Payout, Advertising % of Sales, Advertising Spend, Commission Fee %, Commission Fee, etc.  
- Totals at the bottom provide a rollup across all stores/platforms (e.g. Net Sales 71,528.16; Net Payout 48,325.15).  

### **Filters / Slicers**  
- **Mode (exGST / incGST)** – Toggle between values excluding and including GST.  
- **Platform slicer** – Filter by DoorDash, UberEats, Menulog or view all.  
- **Store Name slicer** – Drill into individual stores.  

This enables dynamic viewing by tax treatment, platform, and store.  

---

## 🗂️ Data Model (Star Schema)

![Data Model](visual%205.png)

### **Dimensional Modelling**
- **Dim_Store** (Store Name)  
- **Dim_Date** (WeekEnd_Sunday)  
- **Dim_Platform** (Platform)  
- **Fact_Payouts_Aggregated** (Main fact table for visuals)

The fact table is linked to each dimension via one-to-many relationships, allowing for flexible slicing & filtering.

### **Supporting Tables**
- **GST_Toggle** – stores the “exGST” and “incGST” toggle selection.  
- **.Measures** – holds all key DAX measures.  
- **Raw Platform Tables** (UberEats_8weeks, Menulog_WeeklyPayments, etc.) – feed into Fact_Payouts_Aggregated.  

This star schema ensures **optimal performance** and **easier measure calculation**.

---

## 📦 Aggregated Fact Table  

I created **Fact_Payouts_Aggregated** using `SUMMARIZECOLUMNS` to pre-aggregate Net Sales, Commission, Advertising, and Net Payout at the **Store + Platform + WeekEnd_Sunday** level (both exGST and incGST).  
This speeds up report rendering because heavy calculations are done once in Power Query/DAX instead of on the fly in visuals.  

```DAX
Fact_Payouts_Aggregated =
SUMMARIZECOLUMNS (
    Fact_Payouts[Store Name],
    Fact_Payouts[Platform],
    Fact_Payouts[WeekEnd_Sunday],
    "NetSales_ex", SUM(Fact_Payouts[NetSales_ex]),
    "NetSales_inc", SUM(Fact_Payouts[NetSales_inc]),
    "Commission_ex", SUM(Fact_Payouts[Commission_ex]),
    "Commission_inc", SUM(Fact_Payouts[Commission_inc]),
    "Advertising_ex", SUM(Fact_Payouts[Advertising_ex]),
    "Advertising_inc", SUM(Fact_Payouts[Advertising_inc]),
    "NetPayout_ex", SUM(Fact_Payouts[NetPayout_ex]),
    "NetPayout_inc", SUM(Fact_Payouts[NetPayout_inc])
)
```

---

## 🧮 Key Measures (DAX)

```DAX
Net Sales =
SWITCH (
    SELECTEDVALUE(GST_Toggle[Mode]),
    "exGST", SUM(Fact_Payouts_Aggregated[NetSales_ex]),
    "incGST", SUM(Fact_Payouts_Aggregated[NetSales_inc])
)

Commission Fee =
SWITCH (
    SELECTEDVALUE(GST_Toggle[Mode]),
    "exGST", SUM(Fact_Payouts_Aggregated[Commission_ex]),
    "incGST", SUM(Fact_Payouts_Aggregated[Commission_inc])
)

Commission Fee % of Sales =
DIVIDE ( [Commission Fee], [Net Sales] )

Advertising Spend =
SWITCH (
    SELECTEDVALUE(GST_Toggle[Mode]),
    "exGST", SUM(Fact_Payouts_Aggregated[Advertising_ex]),
    "incGST", SUM(Fact_Payouts_Aggregated[Advertising_inc])
)

Advertising % of Sales =
DIVIDE ( [Advertising Spend], [Net Sales] )

Net Payout =
SWITCH (
    SELECTEDVALUE(GST_Toggle[Mode]),
    "exGST", SUM(Fact_Payouts_Aggregated[NetPayout_ex]),
    "incGST", SUM(Fact_Payouts_Aggregated[NetPayout_inc])
)

Take Rate % =
DIVIDE ( [Commission Fee] + [Advertising Spend], [Net Sales] )
```

### Dynamic Titles  

```DAX
Title_Advertising =
"Advertising Spend (" &
SWITCH (
    TRUE(),
    SELECTEDVALUE ( GST_Toggle[Mode] ) = "exGST", "ex-GST",
    SELECTEDVALUE ( GST_Toggle[Mode] ) = "incGST", "inc-GST",
    "ex-GST"
) &
") by Platform"
```

(Similar logic used for Commission, Net Payout, and Net Sales.)

---

## 🚀 Value Delivered
- Clear weekly profitability by platform and store.  
- Easy switch between tax views (exGST/incGST).  
- Dynamic titles and slicers for intuitive use.  
- Aggregated fact table ensures performance at scale.  

---

## 📝 Wrap-Up
This Power BI dashboard gives management a clear weekly view of Captain Pizza’s sales and payouts across delivery platforms.  
I built a star schema around a pre-aggregated fact table storing Net Sales, Commission, Advertising and Net Payout at the Store-Platform-Week level.  

All visuals and KPIs are driven by dynamic DAX measures, making the report lightweight and highly responsive. Non-technical users can slice by store or platform and instantly see the impact of commissions and advertising on net payouts — a direct input to pricing and promotional decisions.

---

### 📷 Visuals Included  
- **visual 1.png** – Dashboard Overview
![Dashboard Overview](https://github.com/joseph0327/Power-BI-Captain-Pizza-Sales-Report/blob/b54b5cad5fe8d25b2ff438aab1d0e07d55709e6b/visual%201.png)
- **visual 2.png** – Detailed Table View
![Dashboard Overview](https://github.com/joseph0327/Power-BI-Captain-Pizza-Sales-Report/blob/28e5d889da49a081f81cfe49fd1012c3310e7e63/visual%202.png)
- **visual 5.png** – Data Model  
![Dashboard Overview](https://github.com/joseph0327/Power-BI-Captain-Pizza-Sales-Report/blob/28e5d889da49a081f81cfe49fd1012c3310e7e63/visual%205.png)
