---
title: "Set up AWS budget alert"
date: 2025-12-17 13:34:11
categories: [AWS]
tags: [Cloud]
---

When operating in a cloud environment, it's never not wise to set up a budget alert. A budget alert is basically a system that alerts you when you exceed your budgeted thresholds. You may think you'd be careful on your expense, but I had that confident when I had my hands on Azure Cloud. At the time, I was creating a phishing infrastructure on Azure for my school project with the free credit Azure account. Researching the interet showed me the project will not be that complicated, and I ended up creating a working phishing infrastructure. However, by the time I presented my project to school, I was billed a few hundred dollars for my Azure resource usages. I know I was careless despite my confidence. I totally forgot to check what resources I was using or how long I were using them. Worse of all, I didn't know I needed somethign like budge alert set up for my Azure account. I don't know if Azure has that feature but AWS does. 

So in this post I am going to create a budget alert that's very simple and going save me from spending dollars on unnecssary and careless usuages. 


On the top right of the search bar, search for "budgets" and click the "Budgets" feature.
![Search budget](/images/2025/12-17-AWS-budget-alert-1.png)

For the Budget Setup and Templates, I am going with the default selected plans. I plan to spend the next 6 montsh playing some basic AWS pentest labs, so this "Zero spend budget" should suffice for me. 
![Default budget](/images/2025/12-17-AWS-budget-alert-2.png)

Next, you can put in one or more emails for AWS to notify you if your spending exceeds your budget threshold, which is $0.01.
![email notification](/images/2025/12-17-AWS-budget-alert-3.png)

Now create the budget by pressing the "Create budget" at the end of the bottom corner. 

>Here are a couple of tips to keep costs near $0.00.
1. Terminate, don't stop.
- For services like EC2 instances, if you don't need the instance for a while, terminate (delete) it rather than just stopping it. Stopped instances might still incur charges for associated resources like Amazon EBS storage and Elastic IP Addresses.
2. Monitor Usage
- Make sure you check AWS Billing and Cost Management Console to know that your usuages remain within the free plan limits. 
{: .prompt-tip }

