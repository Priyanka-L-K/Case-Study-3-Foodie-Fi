# 🥑 #3 Foodie-Fi Case Study 🥑

<p align="center">
  <img src="https://8weeksqlchallenge.com/images/case-study-designs/3.png" alt="Foodie-Fi" width="480" />
</p>

---

> 🍳 **Welcome to Foodie-Fi!** The only streaming service where your cravings meet your screen. Imagine Netflix, but *only* for all things FOOD!  

---

## 📋 What’s Cooking?

- [The Story Behind Foodie-Fi](#the-story-behind-foodie-fi)  
- [ERD: Our Recipe for Data](#erd-our-recipe-for-data)  
- [The Secret Ingredients: Data Tables](#the-secret-ingredients-data-tables)  
- [Challenge Questions & Solutions](#challenge-questions--solutions)  
- [Extra Goodies (Resources)](#extra-goodies-resources)  

---

## The Story Behind Foodie-Fi

In 2020, Danny had a delicious idea — why not build a streaming service *just* for food lovers? No cat videos, no dramas — just mouthwatering recipes, cooking shows, and tasty content from around the globe.

He rallied some foodie friends, whipped up Foodie-Fi, and started serving monthly and annual subscriptions like hotcakes!

With data as the secret sauce, Foodie-Fi wanted to slice, dice, and analyze every bite of customer behavior and subscription trends to grow smart and stay fresh.

This case study dishes out the juicy insights with SQL queries that uncover customer journeys, subscription flavors, payment trends, and more.

---

## ERD: Our Recipe for Data

<p align="center">
  <img src="https://github.com/Chisomnwa/8-Week-SQL-Challenge-Case-Study--3-Foodie-Fi/blob/main/Images/ERD.png" alt="Entity Relationship Diagram" width="600" />
</p>

---

## The Secret Ingredients: Data Tables

<details>
  <summary>Click to expand the Tables 🍽️</summary>

### plans Table

| plan_id | plan_name     | price  |
|---------|---------------|--------|
| 0       | trial         | $0     |
| 1       | basic monthly | $9.90  |
| 2       | pro monthly   | $19.90 |
| 3       | pro annual    | $199   |
| 4       | churn         | NULL   |

### subscriptions Table (sample)

| customer_id | plan_id | start_date |
|-------------|---------|------------|
| 1           | 0       | 2020-08-01 |
| 1           | 1       | 2020-08-08 |
| 2           | 0       | 2020-09-20 |
| 2           | 3       | 2020-09-27 |
| ...         | ...     | ...        |

</details>

---

## Case Study Questions & Solutions

Hungry for more? Check out the deep dives here:

- [🍽️ Part A: Customer Journey](https://github.com/Priyanka-L-K/Case-Study-3-Foodie-Fi/blob/main/A.%20Customer%20Journey.md)  
- [📊 Part B: Data Analysis](https://github.com/Priyanka-L-K/Case-Study-3-Foodie-Fi/blob/main/B.%20Data%20Analysis%20Questions.md)  
- [💰 Part C: Payment Insights](https://github.com/Priyanka-L-K/Case-Study-3-Foodie-Fi/blob/main/C.%20Challenge%20Payment%20Question.md)  
- [🚀 Part D: Outside-the-Box Thinking](https://github.com/Priyanka-L-K/Case-Study-3-Foodie-Fi/blob/main/D.%20Outside%20The%20Box%20Questions.md)  

---

## Extra Goodies (Resources)

- [🍔 Medium Article: Deep Dive into Foodie-Fi’s Data]()  
- [📈 Foodie-Fi Dashboard & Key Recommendations]()  

---

<p align="center" style="margin-top: 40px;">  
🔥 Thanks for stopping by Foodie-Fi! Stay hungry, stay curious! 🙌🥰😎  
</p>
