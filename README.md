# Atlassian-Data-Analytics-Project: Jira Sprint Performance & Developer Productivity

## Project Overview

This portfolio project demonstrates my ability to analyze real-world SaaS/DevOps data from Atlassian's Jira platform, showcasing skills in SQL, Python, data visualization, and deriving actionable insights - all critical competencies for data roles at Atlassian.

## Project Components

### 1. Data Understanding & Preparation

**Dataset Description:**
- **jira_issues.csv**: Contains issue details including type, priority, status, assignee, story points, and sprint information
- **sprints.csv**: Contains sprint metadata (name, dates)
- **developers.csv**: Contains developer information (name, team, join date)

**Data Cleaning & Preparation (Python):**

```python
import pandas as pd
import numpy as np
from datetime import datetime

# Load datasets
jira_issues = pd.read_csv('jira_issues.csv')
sprints = pd.read_csv('sprints.csv')
developers = pd.read_csv('developers.csv')

# Convert dates to datetime
jira_issues['created_date'] = pd.to_datetime(jira_issues['created_date'])
jira_issues['resolved_date'] = pd.to_datetime(jira_issues['resolved_date'])
sprints['start_date'] = pd.to_datetime(sprints['start_date'])
sprints['end_date'] = pd.to_datetime(sprints['end_date'])
developers['join_date'] = pd.to_datetime(developers['join_date'])

# Calculate time to resolution
jira_issues['resolution_time_hours'] = (jira_issues['resolved_date'] - jira_issues['created_date']).dt.total_seconds() / 3600

# Merge datasets
merged_data = jira_issues.merge(sprints, on='sprint_id', how='left')
merged_data = merged_data.merge(developers, left_on='assignee_id', right_on='developer_id', how='left')
```

### 2. SQL Analysis

Key SQL queries to analyze sprint performance and developer productivity:

```sql
-- 1. Sprint Completion Rate
SELECT 
    s.sprint_id,
    s.sprint_name,
    COUNT(i.issue_id) AS total_issues,
    SUM(CASE WHEN i.status = 'Done' THEN 1 ELSE 0 END) AS completed_issues,
    ROUND(SUM(CASE WHEN i.status = 'Done' THEN 1 ELSE 0 END) * 100.0 / COUNT(i.issue_id), 2) AS completion_rate
FROM 
    sprints s
LEFT JOIN 
    jira_issues i ON s.sprint_id = i.sprint_id
GROUP BY 
    s.sprint_id, s.sprint_name
ORDER BY 
    s.sprint_id;

-- 2. Developer Productivity (Story Points Completed)
SELECT 
    d.developer_id,
    d.name,
    d.team,
    COUNT(i.issue_id) AS total_tasks,
    SUM(i.story_points) AS total_story_points,
    ROUND(AVG(i.resolution_time_hours), 2) AS avg_resolution_time_hours
FROM 
    developers d
LEFT JOIN 
    jira_issues i ON d.developer_id = i.assignee_id
WHERE 
    i.status = 'Done'
GROUP BY 
    d.developer_id, d.name, d.team
ORDER BY 
    total_story_points DESC;

-- 3. Issue Type Distribution by Priority
SELECT 
    issue_type,
    priority,
    COUNT(*) AS count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM 
    jira_issues
GROUP BY 
    issue_type, priority
ORDER BY 
    issue_type, priority;
```

### 3. Python Analysis & Visualization

**Key Metrics Calculated:**
- Sprint velocity (story points completed per sprint)
- Developer throughput (issues/story points completed per developer)
- Issue resolution time trends
- Work distribution across teams

**Visualizations:**

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Sprint Velocity
sprint_velocity = merged_data[merged_data['status'] == 'Done'].groupby(['sprint_id', 'sprint_name'])['story_points'].sum().reset_index()
plt.figure(figsize=(10, 6))
sns.barplot(x='sprint_name', y='story_points', data=sprint_velocity)
plt.title('Sprint Velocity (Completed Story Points)')
plt.xlabel('Sprint')
plt.ylabel('Story Points Completed')
plt.show()

# Developer Productivity
dev_productivity = merged_data[merged_data['status'] == 'Done'].groupby(['name', 'team'])['story_points'].sum().reset_index()
plt.figure(figsize=(12, 6))
sns.barplot(x='name', y='story_points', hue='team', data=dev_productivity)
plt.title('Developer Productivity by Story Points Completed')
plt.xlabel('Developer')
plt.ylabel('Story Points Completed')
plt.xticks(rotation=45)
plt.show()

# Resolution Time by Issue Type
plt.figure(figsize=(10, 6))
sns.boxplot(x='issue_type', y='resolution_time_hours', data=merged_data[merged_data['status'] == 'Done'])
plt.title('Resolution Time Distribution by Issue Type')
plt.xlabel('Issue Type')
plt.ylabel('Resolution Time (hours)')
plt.yscale('log')  # Using log scale due to outliers
plt.show()
```

![sprint velocity trend](https://github.com/user-attachments/assets/15a2ef14-a346-4e59-ac0b-58f087503382)

![cycle time distribution](https://github.com/user-attachments/assets/03441d60-dc54-46c9-85b8-9af10b1baace)

![age of unfinished work](https://github.com/user-attachments/assets/ef516ebf-59c1-402e-a01a-aa1ada19ab1f)

![devloper throughput](https://github.com/user-attachments/assets/9dc2a88d-7ba0-45cb-8aca-ee9dd6dc17f6)

![time spent in diffrent workflow states](https://github.com/user-attachments/assets/2d3ed9fc-a307-434b-99a0-4536ea613aa4)


### 4. Tableau Dashboard

Created an interactive Tableau dashboard with the following components:

1. **Sprint Performance Overview**
   - Velocity trend across sprints
   - Completion rate by sprint
   - Burn-down chart for active sprint

2. **Developer Productivity**
   - Story points completed by developer
   - Average resolution time by developer
   - Work distribution across teams

3. **Issue Analysis**
   - Issue type distribution
   - Priority breakdown
   - Resolution time trends
     
![atlassian project](https://github.com/user-attachments/assets/20afb006-86d3-44f2-b101-9100324cd583)


**Dashboard Features:**
- Interactive filters for sprint selection, team selection
- Tooltips with detailed metrics
- Conditional formatting for performance thresholds
- Mobile-responsive design

### 5. Key Insights & Recommendations

**Findings:**
1. Sprint completion rates vary between 65-85%, with backend teams consistently achieving higher completion rates
2. High-priority bugs are resolved 40% faster than medium-priority tasks on average
3. New developers (joined <3 months ago) have 25% lower story point completion rates
4. Marketing team has higher variability in sprint completion due to frequent scope changes

**Actionable Recommendations:**
1. Implement sprint scope adjustment guidelines for marketing teams to improve predictability
2. Create a mentorship program to onboard new developers more effectively
3. Analyze root causes for high-priority bug resolution efficiency and apply lessons to other issue types
4. Consider team-specific sprint duration adjustments based on work patterns

## Project Value

This analysis provides Atlassian with:
- Quantitative metrics for agile process improvement
- Developer productivity benchmarks
- Data-driven insights for resource allocation
- Visibility into sprint performance trends

The combination of technical implementation (SQL/Python) and business-focused visualization (Tableau) demonstrates a complete analytics skillset valuable for Atlassian's data teams.

---

**Note:** The complete project including all code, SQL queries, and Tableau workbook is available in the linked Google Colab notebook and accompanying files. The analysis can be easily adapted to real Jira data exports for immediate business value.
You can find it here.
https://colab.research.google.com/drive/1GFCXBP7L3gNNqEJam4mipRalIsLXpqig?usp=sharing
