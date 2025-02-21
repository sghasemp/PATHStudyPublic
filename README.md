---
title: "Rmaincode"
output: html_document
date: "2023-12-18"
---

```{r}

#Library

rm(list = ls())
cat("\014")  # Clears the console



library(dplyr)
library(ggplot2)
library(mediation)
library(car)
library(emmeans)

```


```{r}
#Read the data

ECfinal<- read.csv("")

```

SNAP /WIC and food insecurity variables:
```{r}
#SNAP
ECfinal$SNAP <- ifelse(grepl("SNAP", ECfinal$CNP4, fixed = TRUE), 1, 0)
summary(as.factor(ECfinal$SNAP))

# WIC
ECfinal$WIC <- ifelse(grepl("WIC", ECfinal$CNP4, fixed = TRUE), 1, 0)
summary(as.factor(ECfinal$WIC))
#Create a new binary column 'WIC_or_SNAP' based on the presence of either "WIC" or "SNAP"
ECfinal$WIC_or_SNAP <- ifelse(grepl("WIC", ECfinal$CNP4, fixed = TRUE) | 
                              grepl("SNAP", ECfinal$CNP4, fixed = TRUE), 1, 0)

# Summary to check the distribution of WIC_or_SNAP
summary(as.factor(ECfinal$WIC_or_SNAP))

#total number of people receiving either WIC or SNAP
WS <- sum(ECfinal$WIC_or_SNAP)
WS

# food insecurity
ECfinal <- ECfinal %>%
  mutate(CNP1_code = case_when(
           grepl("Often", CNP1, fixed = TRUE) ~ 1,
           grepl("Sometimes", CNP1, fixed = TRUE) ~ 1,
           grepl("Never", CNP1, fixed = TRUE) ~ 0,
           grepl("I don't know", CNP1, fixed = TRUE) ~ 0,
           TRUE ~ NA_real_),
         CNP2_code = case_when(
           grepl("Often", CNP2, fixed = TRUE) ~ 1,
           grepl("Sometimes", CNP2, fixed = TRUE) ~ 1,
           grepl("Never", CNP2, fixed = TRUE) ~ 0,
           grepl("I don't know", CNP2, fixed = TRUE) ~ 0,
           TRUE ~ NA_real_),
         CNP3_code = case_when(
           grepl("Yes", CNP3, fixed = TRUE) ~ 1,
           grepl("No", CNP3, fixed = TRUE) ~ 0,
           grepl("I don't know", CNP3, fixed = TRUE) ~ 0,
           TRUE ~ NA_real_))

# Calculate raw score for food insecurity
ECfinal <- ECfinal %>%
  mutate(security = CNP1_code + CNP2_code + CNP3_code)



# Determine food insecurity status 
ECfinal <- ECfinal %>%
  mutate(food_insecure = ifelse(security >= 1, 1, 0))

# Calculate the percentage of food insecure participants within the SNAP/WIC group
snap_wic_insecure <- ECfinal %>%
  filter(WIC_or_SNAP == 1) %>%
  summarise(percentage_insecure = mean(food_insecure, na.rm = TRUE) * 100)

snap_wic_insecure



```





Picky eating- Parent control-encouragement emotional and instrumental variable:

```{r}
# Define the mapping from text responses to numeric values
EClist <- c("Always" = 5, "Often" = 4, "Sometimes" =3 , "Rarely" = 2, "Never" = 1 )

# List the question columns for each category based on the names you provided
control_questions <- c("PFSQ1","PFSQ5","PFSQ14","PFSQ16", "PFSQ17", "PFSQ11", "PFSQ20","PFSQ23", "PFSQ24", "PFSQ26")
emotional_questions <- c("PFSQ25", "PFSQ21", "PFSQ15", "PFSQ13", "PFSQ2")
instrumental_questions <- c("PFSQ7", "PFSQ9", "PFSQ18", "PFSQ22")
encouragemental_questions <- c("PFSQ19","PFSQ27","PFSQ10","PFSQ12", "PFSQ8", "PFSQ6", "PFSQ4","PFSQ3")
picky_questions <- c("CEBQ32", "CEBQ24", "CEBQ16", "CEBQ10", "CEBQ7")

# Convert text responses to numeric values and calculate mean scores for control, emotional eating, and picky eating
ECfinal <- ECfinal %>%
  mutate(
    across(all_of(control_questions), ~ as.numeric(EClist[.x]), .names = "numeric_{.col}"),
    across(all_of(emotional_questions), ~ as.numeric(EClist[.x]), .names = "numeric_{.col}"),
    across(all_of(encouragemental_questions), ~ as.numeric(EClist[.x]), .names = "numeric_{.col}"),
    across(all_of(instrumental_questions), ~ as.numeric(EClist[.x]), .names = "numeric_{.col}"),
    across(all_of(picky_questions), ~ as.numeric(EClist[.x]), .names = "numeric_{.col}")
  ) %>%
  rowwise() %>%
  mutate(
    Control_Score = mean(c(c_across(starts_with("numeric_PFSQ5")),
                           c_across(starts_with("numeric_PFSQ16")),
                           c_across(starts_with("numeric_PFSQ11")),
                           c_across(starts_with("numeric_PFSQ17")),
                           c_across(starts_with("numeric_PFSQ1")),
                           c_across(starts_with("numeric_PFSQ14")),
                           c_across(starts_with("numeric_PFSQ23")),
                           c_across(starts_with("numeric_PFSQ20")),
                           c_across(starts_with("numeric_PFSQ24")),
                           c_across(starts_with("numeric_PFSQ26"))), na.rm = TRUE),
    Instru_Score = mean(c(c_across(starts_with("numeric_PFSQ7")),
                           c_across(starts_with("numeric_PFSQ9")),
                           c_across(starts_with("numeric_PFSQ18")),
                           c_across(starts_with("numeric_PFSQ22"))), na.rm = TRUE),
         Encourage_Score = mean(c(c_across(starts_with("numeric_PFSQ3")),
                           c_across(starts_with("numeric_PFSQ6")),
                           c_across(starts_with("numeric_PFSQ4")),
                           c_across(starts_with("numeric_PFSQ8")),
                           c_across(starts_with("numeric_PFSQ10")),
                           c_across(starts_with("numeric_PFSQ12")),
                           c_across(starts_with("numeric_PFSQ19")),
                           c_across(starts_with("numeric_PFSQ27"))), na.rm = TRUE),
    Emotional_Score = mean(c(c_across(starts_with("numeric_PFSQ25")),
                             c_across(starts_with("numeric_PFSQ21")),
                             c_across(starts_with("numeric_PFSQ15")),
                             c_across(starts_with("numeric_PFSQ13")),
                             c_across(starts_with("numeric_PFSQ2"))), na.rm = TRUE),
    Picky_Score = mean(c(c_across(starts_with("numeric_CEBQ7")),
                             c_across(starts_with("numeric_CEBQ10")),
                             c_across(starts_with("numeric_CEBQ16")),
                             c_across(starts_with("numeric_CEBQ24")),
                             c_across(starts_with("numeric_CEBQ32"))), na.rm = TRUE)
  ) %>%
  ungroup()

```


Demographic variables:


```{r}
#Household

ECfinal$household <- as.numeric(as.character(ECfinal$household))
#childs number
ECfinal$num_children <- as.numeric(as.character(ECfinal$num_children))
#Relationship
relationship_map <- c("Never married" = 5, 
                      "Married" = 1, 
                      "Divorced" = 2, 
                      "Separated" = 3, 
                      "Widowed" = 4)

# Convert text responses to numeric values for Relationship status
ECfinal <- ECfinal %>%
  mutate(Relationship = relationship_map[relationship])


#Race 
race_map <- c(
  "White, for example, German, Irish, English, Italian, Lebanese, Egyptian, etc." = 1,
  "Vietnamese" = 2,
  "Other Asian" = 3,
  "Native Hawaiian" = 4,
  "Japanese" = 5,
  "Chinese" = 6,
  "Black or African American" = 7,
  "Asian Indian" = 8,
  "Another race" = 9,
  "American Indian or Alaska Native, for example, Navajo Nation, Blackfeet Tribe, Mayan, Aztec, Native Village of Barrow Inupiat Traditional Government, Nome Eskimo Community, etc." = 10,
  "Not Available" = NA,  # Assign NA or a specific code if treating as a separate category
  "Other" = 11  # Grouping all other non-standard responses under 'Other'
)

# Apply the mapping, including handling for 'Not Available' and other responses
ECfinal <- ECfinal %>%
  mutate(Race = ifelse(race %in% names(race_map), race_map[race], race_map["Other"]))
education_map <- c("Some high school" = 1, 
                   "High school degree or GED" = 2, 
                   "Some college or associates degree" = 3, 
                   "4 years college degree" = 4, 
                   "Advanced degree" = 5)

# Parent Education
ECfinal <- ECfinal %>%
  mutate(Education = education_map[ed])
#Household Income
income_map <- c("Less than $20,000" = 1, 
                "$20,000 - $39,999" = 2, 
                "$40,000 - $49,999" = 3, 
                "$50,000 - $74,999" = 4, 
                "$75,000 - $99,999" = 5, 
                "More than $100,000" = 6)
#Convert text responses to numeric values and calculate mean scores for Income
ECfinal <- ECfinal %>%
  mutate(Income = income_map[income])

# Parents Role
relation_map <- c("Father" = 1, "Mother" = 2, "Other" = 3)

# Assuming 'com2' is your data frame and 'relation' is the column with these responses
ECfinal <- ECfinal %>%
  mutate(Role = relation_map[parent])

# Child_sex
sex_map <- c("Male" = 1, "Female" = 2)

# Assuming 'com2' is your data frame and 'sex' is the column with these responses
ECfinal <- ECfinal %>%
  mutate(Sex = sex_map[sex])


#Child_Age
ECfinal$Age <- as.numeric(as.character(ECfinal$Age))

# ethnicity
ethnicity_map <- c("Yes" = 1, "No" = 2)

ECfinal <- ECfinal %>%
  mutate(Ethnicity = ethnicity_map[ethnicity])

```




T-tests: food insecurity, parental feeding styls and picky eating in the SNAP/Non SNAP families

```{r}
#T-test to assess difference between mean picky eating in the SNAP and non SNAP

# Extract control scores for SNAP/WIC participants
pick_scores_snap2 = ECfinal$Picky_Score[ECfinal$WIC_or_SNAP == "1"]

# Extract control scores for non-SNAP/WIC participants
pick_scores_non_snap2 = ECfinal$Picky_Score[ECfinal$WIC_or_SNAP == "0"]

# Perform a two-sided t-test between the two groups
t_test_result2 = t.test(pick_scores_snap2, pick_scores_non_snap2, alternative = "two.sided")

print(t_test_result2)
difference_in_means1 = 3.426974 - 3.078351 
t_statistic1 = 4.1806
SE1 = difference_in_means1 / t_statistic1

print(SE1)





#to assess difference between mean control feeding style in the SNAP and non SNAP
#  control scores for SNAP/WIC participants
control_scores_snap = ECfinal$Control_Score[ECfinal$WIC_or_SNAP == "1"]

# control scores for non-SNAP/WIC participants
control_scores_non_snap = ECfinal$Control_Score[ECfinal$WIC_or_SNAP == "0"]

# Perform a two-sided t-test between the two groups
t_test_result = t.test(control_scores_snap, control_scores_non_snap, alternative = "two.sided")

print(t_test_result)

# Given values
difference_in_means = 3.490132 - 3.195931
t_statistic = 3.7982
SE = difference_in_means / t_statistic
print(SE)






# T-test to assess difference between mean emotional feeding style in the SNAP and non SNAP
#control scores for SNAP/WIC participants
Emotional_scores_snap = ECfinal$Emotional_Score[ECfinal$WIC_or_SNAP == "1"]

# control scores for non-SNAP/WIC participants
Emotional_scores_non_snap = ECfinal$Emotional_Score[ECfinal$WIC_or_SNAP == "0"]

# Perform a two-sided t-test between the two groups
t_test_result2 = t.test(Emotional_scores_snap, Emotional_scores_non_snap, alternative = "two.sided")

print(t_test_result2)

# Given values
difference_in_means2 = 3.358300 - 2.697528 
t_statistic2 = 6.5879
SE2 = difference_in_means2 / t_statistic2
print(SE2)






#T-test to assess difference between mean instrumental feeding style in the SNAP and non SNAP
#  control scores for SNAP/WIC participants
Instru_scores_snap = ECfinal$Instru_Score[ECfinal$WIC_or_SNAP == "1"]

# control scores for non-SNAP/WIC participants
Instru_scores_non_snap = ECfinal$Instru_Score[ECfinal$WIC_or_SNAP == "0"]

# Perform a two-sided t-test between the two groups
t_test_result3 = t.test(Instru_scores_snap, Instru_scores_non_snap, alternative = "two.sided")

print(t_test_result3)

# Given values
difference_in_means3 = 3.223684 - 2.402062 
t_statistic3 = 5.6865
SE3 = difference_in_means3 / t_statistic3
print(SE3)







#T-test to assess difference between mean encouragement feeding style in the SNAP and non SNAP
#   for SNAP/WIC participants
Encourage_Score_snap = ECfinal$Encourage_Score[ECfinal$WIC_or_SNAP == "1"]

#  for non-SNAP/WIC participants
Encourage_Score_non_snap = ECfinal$Encourage_Score[ECfinal$WIC_or_SNAP == "0"]

# Perform a two-sided t-test between the two groups
t_test_result4 = t.test(Encourage_Score_snap, Encourage_Score_non_snap, alternative = "two.sided")

print(t_test_result4)

# Given values
difference_in_means4 = 3.975094 - 4.094072 
t_statistic4 = -1.5567
SE4 = difference_in_means4 / t_statistic4
print(SE4)




#t-test food security between SNAP and NON SNAP
 
food_insecurity_scores_snap <- ECfinal$security [ECfinal$WIC_or_SNAP == 1]

# Food insecurity scores for non-SNAP/WIC participants
food_insecurity_scores_non_snap <- ECfinal$security [ECfinal$WIC_or_SNAP == 0]

# Perform a two-sided t-test between the two groups
t_test_result_food_insecurity <- t.test(food_insecurity_scores_snap, food_insecurity_scores_non_snap, alternative = "two.sided")

print(t_test_result_food_insecurity)


# Given values
difference_in_means = 1.3000000 - 0.4111111 
t_statistic = 5.4612
SEe = difference_in_means / t_statistic
print(SEe)



```

Violin plot for T tests.
```{r}


#Violin plot for difference between mean control feeding style in the SNAP and non SNAP
# Create a combined data frame for plotting
combined_scores <- data.frame(
  score = c(control_scores_snap, control_scores_non_snap),
  group = factor(c(rep("SNAP/WIC", length(control_scores_snap)), 
                   rep("Non-SNAP/WIC", length(control_scores_non_snap))))
)

# Violin plot with mean line
p1<- ggplot(combined_scores, aes(x = group, y = score, fill = group)) +
  geom_violin(trim = FALSE) + # Draw violin plot
  stat_summary(geom="crossbar", width=0.5) + # Add mean line
  scale_fill_manual(values = c("lightblue", "salmon")) + # Colors
  labs(title = "Controlling feeding style by SNAP/WIC Status",
       y = "Controlling feeding style",
       x = "SNAP/WIC") +
  theme_minimal() # Use a minimal theme for a cleaner look

# Save the plot
ggsave("controlling_violin.png", dpi = 300)








# plot for difference between meanpicky eating in the SNAP and non SNAP
# Create a combined data frame for plotting
combined_scores1 <- data.frame(
  score = c(pick_scores_snap2, pick_scores_non_snap2),
  group = factor(c(rep("SNAP/WIC", length(pick_scores_snap2)), 
                   rep("Non-SNAP/WIC", length(pick_scores_non_snap2))))
)

# Create the violin plot
p2 <- ggplot(combined_scores1, aes(x = group, y = score, fill = group)) +
  geom_violin(trim = FALSE) + # Draw violin plot
  stat_summary(geom="crossbar", width=0.5) + # Add mean line
  scale_fill_manual(values = c("lightblue", "salmon")) + 
  labs(title = "Picky eating by SNAP/WIC Status",
       y = "Picky eating",
       x = "SNAP/WIC") +
  theme_minimal() 

# Save 
ggsave("pickyeating_violin.png", dpi = 300)





#Violin plot for difference between mean Emotional feeding style in the SNAP and non SNAP
combined_scores <- data.frame(
  score = c(Emotional_scores_snap, Emotional_scores_non_snap),
  group = factor(c(rep("SNAP/WIC", length(Emotional_scores_snap)), 
                   rep("Non-SNAP/WIC", length(Emotional_scores_non_snap))))
)

# Violin plot with mean line
p1<- ggplot(combined_scores, aes(x = group, y = score, fill = group)) +
  geom_violin(trim = FALSE) + # Draw violin plot
  stat_summary(geom="crossbar", width=0.5) + # Add mean line
  scale_fill_manual(values = c("lightblue", "salmon")) + # Colors
  labs(title = "Emotional feeding style by SNAP/WIC Status",
       y = "Emotional feeding style",
       x = "SNAP/WIC") +
  theme_minimal() # Use a minimal theme for a cleaner look

# Save the plot
ggsave("emotional_violin.png", dpi = 300)





#Violin plot for difference between mean Encouragement feeding style in the SNAP and non SNAP
combined_scores5 <- data.frame(
  score = c(Encourage_Score_snap, Encourage_Score_non_snap),
  group = factor(c(rep("SNAP/WIC", length(Encourage_Score_snap)), 
                   rep("Non-SNAP/WIC", length(Encourage_Score_non_snap))))
)

# Violin plot with mean line
p1<- ggplot(combined_scores5, aes(x = group, y = score, fill = group)) +
  geom_violin(trim = FALSE) + # Draw violin plot
  stat_summary(geom="crossbar", width=0.5) + # Add mean line
  scale_fill_manual(values = c("lightblue", "salmon")) + # Colors
  labs(title = "Encouragement feeding style by SNAP/WIC Status",
       y = "Encouragement feeding style",
       x = "SNAP/WIC") +
  theme_minimal() # Use a minimal theme for a cleaner look

# Save the plot
ggsave("encouragement_violin.png", dpi = 300)






#Violin plot for difference between mean Instrumental feeding style in the SNAP and non SNAP
combined_scores3 <- data.frame(
  score = c(Instru_scores_snap, Instru_scores_non_snap),
  group = factor(c(rep("SNAP/WIC", length(Instru_scores_snap)), 
                   rep("Non-SNAP/WIC", length(Instru_scores_non_snap))))
)

# Violin plot with mean line
p1<- ggplot(combined_scores3, aes(x = group, y = score, fill = group)) +
  geom_violin(trim = FALSE) + # Draw violin plot
  stat_summary(geom="crossbar", width=0.5) + # Add mean line
  scale_fill_manual(values = c("lightblue", "salmon")) + # Colors
  labs(title = "Instrumental feeding style by SNAP/WIC Status",
       y = "Instrumental feeding style",
       x = "SNAP/WIC") +
  theme_minimal() # Use a minimal theme for a cleaner look

# Save the plot
ggsave("Instrumental_violin.png", dpi = 300)





#Violin plot for difference between mean security in the SNAP and non SNAP
# Create a combined data frame for plotting
combined_scores4 <- data.frame(
  score = c(food_insecurity_scores_snap, food_insecurity_scores_non_snap),
  group = factor(c(rep("SNAP/WIC", length(food_insecurity_scores_snap)), 
                   rep("Non-SNAP/WIC", length(food_insecurity_scores_non_snap))))
)

# Violin plot with mean line
p1<- ggplot(combined_scores4, aes(x = group, y = score, fill = group)) +
  geom_violin(trim = FALSE) + # Draw violin plot
  stat_summary(geom="crossbar", width=0.5) + # Add mean line
  scale_fill_manual(values = c("lightblue", "salmon")) + # Colors
  labs(title = "Food Security by SNAP/WIC Status",
       y = "Food Security",
       x = "SNAP/WIC") +
  theme_minimal() # Use a minimal theme for a cleaner look

# Save the plot
ggsave("security_violin.png", dpi = 300)


```




 













I change Picky eating to categorical Data, LOW, MED, HIGH

```{r}
library(dplyr)
# Categorize Picky_Score into three categories: Low, Medium, High
ECfinal <- ECfinal %>%
  mutate(
    Picky_Category = cut(Picky_Score,
                         breaks = quantile(Picky_Score, probs = c(0, 0.33, 0.66, 1), na.rm = TRUE),
                         labels = c("Low", "Medium", "High"),
                         include.lowest = TRUE)
  )

```





Assess the interaction of  SNAP and feeding style on picky eating , and moderation effect of SNAP 
```{r}
#1.Assess the interaction of  SNAP and controlling  feeding style on picky eating

outcome_model<-lm(Picky_Score ~ Control_Score * WIC_or_SNAP +
                      Sex +Age+ Income + Education + Role + household + Relationship + num_children + Race+ Ethnicity+ security, data = ECfinal)
summary(outcome_model)

plot(outcome_model)

#assess moderation of SNAP/WIC on controlling feeding styls and picky eating
emtrends(outcome_model, pairwise ~ WIC_or_SNAP, var = "Control_Score")
emmip(outcome_model, WIC_or_SNAP ~ Control_Score, cov.reduce = range) 
 
 
#Check Multicollinearity for the interaction of  SNAP and controlling  feeding style on picky eating
numeric_predictors <- c("Control_Score", "Sex", "Age", "Relationship", "Income", "Education", "Role", "household","Race", "num_children","Ethnicity","security")
# Now run the linear model again
multicollinearity_model <- lm(ECfinal$Picky_Score ~ ., data = ECfinal[numeric_predictors])
# Calculate VIF
vif_values <- vif(multicollinearity_model)
print(vif_values)




 
 
#2.Assess the interaction of  SNAP and Emotional  feeding style on picky eating
outcome_model2 <- lm(Picky_Score ~ Emotional_Score * WIC_or_SNAP + 
              Sex +Age + Income + Education + Role + household+ Relationship +num_children+Race +Ethnicity+security , data = ECfinal)

summary(outcome_model2)


#ASESS MODERATION EFFECT OF SNAP/WIC in the EMOTIONAL feeding style and picky eating 
 emtrends(outcome_model2, pairwise ~ WIC_or_SNAP, var = "Emotional_Score")
emmip(outcome_model2, WIC_or_SNAP ~ Emotional_Score, cov.reduce = range) 
 
#Check Multicollinearity EFFECT OF SNAP/WIC in the EMOTIONAL group
numeric_predictors2 <- c("Emotional_Score", "Sex", "Age", "Relationship", "Income","num_children", "Education", "Role","Race", "household","Ethnicity","security")
# Now run the linear model again
multicollinearity_model2 <- lm(ECfinal$Picky_Score ~ ., data = ECfinal[numeric_predictors2])
# Calculate VIF
vif_values2 <- vif(multicollinearity_model2)
print(vif_values2)



#3.Assess the effect of SNAP and  encouragement feeding style on picky eating
outcome_model3 <- lm(Picky_Score ~ Encourage_Score * WIC_or_SNAP + 
              Sex +Age + Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security , data = ECfinal)

summary(outcome_model3)


#ASESS MODERATION EFFECT OF SNAP/WIC in encouragment feeding style and picky eating 
 emtrends(outcome_model3, pairwise ~ WIC_or_SNAP, var = "Encourage_Score")
emmip(outcome_model3, WIC_or_SNAP ~ Encourage_Score, cov.reduce = range)


#Check Multicollinearity for effect of SNAP on encouragement feeding style
numeric_predictors3 <- c("Encourage_Score","Age", "Sex", "Relationship", "Income","num_children", "Education", "Role","Race", "household","Ethnicity","security")
# Now run the linear model again
multicollinearity_model3 <- lm(ECfinal$Picky_Score ~ ., data = ECfinal[numeric_predictors3])
# Calculate VIF
vif_values3 <- vif(multicollinearity_model3)
print(vif_values3)



#4.Assess the interaction of  SNAP and instrumental feeding style on picky eating 
outcome_model4 <- lm(Picky_Score ~ Instru_Score * WIC_or_SNAP + 
              Sex +Age + Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security , data = ECfinal)
summary(outcome_model4)

#ASESS MODERATION EFFECT OF SNAP/WIC in the instrumental group
emtrends(outcome_model4, pairwise ~ WIC_or_SNAP, var = "Instru_Score")

#Check Multicollinearity for the interaction of  SNAP and instrumental feeding style on picky eating
numeric_predictors4 <- c("Instru_Score", "Sex", "Relationship", "Income","num_children", "Education", "Role","Race", "household","security", "Ethnicity")
# Now run the linear model again
multicollinearity_model4 <- lm(ECfinal$Picky_Score ~ ., data = ECfinal[numeric_predictors4])

# Calculate VIF
vif_values4 <- vif(multicollinearity_model4)
print(vif_values4)
```




Diagram for the relationship between parental feeding style , picky eating in SNAP/non SNAP 



```{r}

#Controlling feeding style and picky eating in SNAP and nonSNAP
# Scatter plot with separate regression lines for SNAP 0 and SNAP 1
pp1<- ggplot(ECfinal, aes(x = Control_Score, y = Picky_Score, color = factor(WIC_or_SNAP))) +
  geom_point() +  # Add points
  geom_smooth(method = "lm", se = TRUE, aes(group = factor(WIC_or_SNAP))) +  # Add separate regression lines for each SNAP group
  labs(title = "Control Feeding Style vs. Picky Eating by SNAP",
       x = "Control Feeding Style ", y = "Picky Eating",
       color = "WIC or SNAP Status") +
  theme_minimal() +
  scale_color_manual(values = c("blue", "red"),
                    labels = c("0" = "No WIC or SNAP", "1" = "WIC or SNAP") )  # Specify colors for SNAP 0 and SNAP 1
ggsave("m1.png", dpi = 300)




#Emotional feeding style in SNAP and non SNAP

# Scatter plot with separate regression lines for SNAP 0 and SNAP 1
pp2<- ggplot(ECfinal, aes(x = Emotional_Score, y = Picky_Score, color = factor(WIC_or_SNAP))) +
  geom_point() +  # Add points
  geom_smooth(method = "lm", se = TRUE, aes(group = factor(WIC_or_SNAP))) +  # Add separate regression lines for each SNAP group
  labs(title = "Emotional Feeding Style vs. Picky Eating by SNAP",
       x = "Emotional Feeding Style ", y = "Picky Eating",
       color = "WIC or SNAP Status") +
  theme_minimal() +
  scale_color_manual(values = c("blue", "red"),
                    labels = c("0" = "No WIC or SNAP", "1" = "WIC or SNAP") )  # Specify colors for SNAP 0 and SNAP 1

ggsave("m2.png", dpi = 300)






#Diagram for the  effect of SNAP and  encouragement feeding style on picky eating
# Scatter plot with separate regression lines for SNAP 0 and SNAP 1
ggplot(ECfinal, aes(x = Encourage_Score, y = Picky_Score, color = factor(WIC_or_SNAP))) +
  geom_point() +  # Add points
  geom_smooth(method = "lm", se = TRUE, aes(group = factor(WIC_or_SNAP))) +  # Add separate regression lines for each SNAP group
  labs(title = "Encouragement Feeding Style vs. Picky Eating by SNAP",
       x = "Encouragement Feeding Style ", y = "Picky Eating",
       color = "WIC or SNAP Status") +
  theme_minimal() +
  scale_color_manual(values = c("blue", "red"),
                    labels = c("0" = "No WIC or SNAP", "1" = "WIC or SNAP") )  # Specify colors for SNAP 0 and SNAP 1

ggsave("m3.png", dpi = 300)





#Diagram for the interaction of  SNAP and instrumental feeding style on picky eating
# Scatter plot with separate regression lines for SNAP 0 and SNAP 1
pp4<- ggplot(ECfinal, aes(x = Instru_Score, y = Picky_Score, color = factor(WIC_or_SNAP))) +
  geom_point() +  # Add points
  geom_smooth(method = "lm", se = TRUE, aes(group = factor(WIC_or_SNAP))) +  # Add separate regression lines for each SNAP group
  labs(title = "Instrumental Feeding Style vs. Picky Eating by SNAP",
       x = "Instrumental Feeding Style ", y = "Picky Eating",
       color = "WIC or SNAP Status") +
  theme_minimal() +
  scale_color_manual(values = c("blue", "red"),
                    labels = c("0" = "No WIC or SNAP", "1" = "WIC or SNAP") )  # Specify colors for SNAP 0 and SNAP 1


ggsave("m4.png", dpi = 300)

```


Facet Plot to shows different between picky eating in parental feeding styls in the SNAP and NON SNAP group

```{r}

#Facet Plot to shows different between picky eating in Controlling feeding styls in the SNAP and NON SNAP group
# Create the plot
ggplot(ECfinal, aes(x = Control_Score, y = Picky_Score)) +
  geom_point(color = "black") +  # Set scatter points to red
  facet_wrap(~WIC_or_SNAP, scales = "free_y", 
             labeller = labeller(WIC_or_SNAP = c('0' = "Non-SNAP/WIC", '1' = "SNAP/WIC"))) +
  labs(title = "Comparison of Picky Eating in the Controlling Feeding Style by SNAP/WIC Status",
       x = "Controlling Feeding Style", 
       y = "Picky Eating") +
  theme_minimal() +
  theme(
    panel.background = element_rect(fill = "aliceblue"),  # Set background color for the plot area
    strip.text.x = element_text(size = 12, color = "black", face = "bold"),
    panel.spacing = unit(1, "lines"),  # Adjust spacing between facets if needed
    panel.border = element_rect(colour = "aliceblue", fill=NA, size=1),  # Add border around each panel
  )

# Save the plot
ggsave("picky_controlling_facet_plot.png", width = 8, height = 4, dpi = 300)









#Facet Plot to shows different between picky eating in emotional feeding style in the SNAP and NON SNAP group
# Create the plot
ggplot(ECfinal, aes(x = Emotional_Score, y = Picky_Score)) +
  geom_point(color = "black") +  # Set scatter points to red
  facet_wrap(~WIC_or_SNAP, scales = "free_y", 
             labeller = labeller(WIC_or_SNAP = c('0' = "Non-SNAP/WIC", '1' = "SNAP/WIC"))) +
  labs(title = "Comparison of Picky Eating in the Emotional Feeding Style by SNAP/WIC Status",
       x = "Emotional Feeding Style", 
       y = "Picky Eating") +
  theme_minimal() +
  theme(
    panel.background = element_rect(fill = "aliceblue"),  # Set background color for the plot area
    strip.text.x = element_text(size = 12, color = "black", face = "bold"),
    panel.spacing = unit(1, "lines"),  # Adjust spacing between facets if needed
    panel.border = element_rect(colour = "aliceblue", fill=NA, size=1),  # Add border around each panel
  )
# Save the plot
ggsave("picky_emotional_facet_plot.png", width = 8, height = 4, dpi = 300)





#Facet Plot to shows different between picky eating in instrumental feeding style in the SNAP and NON SNAP group
# Create the plot
ggplot(ECfinal, aes(x = Instru_Score, y = Picky_Score)) +
  geom_point(color = "black") +  # Set scatter points to red
  facet_wrap(~WIC_or_SNAP, scales = "free_y", 
             labeller = labeller(WIC_or_SNAP = c('0' = "Non-SNAP/WIC", '1' = "SNAP/WIC"))) +
  labs(title = "Comparison of Picky Eating in the Instrumental Feeding Style by SNAP/WIC Status",
       x = "Instrumental Feeding Style", 
       y = "Picky Eating") +
  theme_minimal() +
  theme(
    panel.background = element_rect(fill = "aliceblue"),  # Set background color for the plot area
    strip.text.x = element_text(size = 12, color = "black", face = "bold"),
    panel.spacing = unit(1, "lines"),  # Adjust spacing between facets if needed
    panel.border = element_rect(colour = "aliceblue", fill=NA, size=1),  # Add border around each panel
  )

# Save the plot
ggsave("picky_instrumental_facet_plot.png", width = 8, height = 4, dpi = 300)

#Facet Plot to shows different between picky eating in encouragement feeding style in the SNAP and NON SNAP group
# Create the plot
ggplot(ECfinal, aes(x = Encourage_Score, y = Picky_Score)) +
  geom_point(color = "black") +  # Set scatter points to red
  facet_wrap(~WIC_or_SNAP, scales = "free_y", 
             labeller = labeller(WIC_or_SNAP = c('0' = "Non-SNAP/WIC", '1' = "SNAP/WIC"))) +
  labs(title = "Comparison of Picky Eating in the Encouragement Feeding Style by SNAP/WIC Status",
       x = "Encouragement Feeding Style", 
       y = "Picky Eating") +
  theme_minimal() +
  theme(
    panel.background = element_rect(fill = "aliceblue"),  # Set background color for the plot area
    strip.text.x = element_text(size = 12, color = "black", face = "bold"),
    panel.spacing = unit(1, "lines"),  # Adjust spacing between facets if needed
    panel.border = element_rect(colour = "aliceblue", fill=NA, size=1),  # Add border around each panel
  )

# Save the plot
ggsave("picky_encouragement_facet_plot.png", width = 8, height = 4, dpi = 300)
```





****************relationship between parental feeding style, child picky eating and BMI*****************************8

```{r}
#library

library(tidyr)
library("ggplot2")
library("childsds")
library("anthro")
library(car)
library("dplyr")
library(nnet)
library(ggplot2)
library(dplyr)
library(ggrepel)
library(ggrepel)




```


Read the data and merge the measurments with ECfinal


```{r}
 #read data
cmeasure<- read.csv("")


#merged ECfinal with measurment data based on RandomID
Data <- merge(cmeasure, ECfinal, by = "RandomID")


```

BMI Percentile

```{r}
#BMI Percentile

Data$BMIpercentile <- sds(
  value = Data$BMI,
  age = Data$Age,
  sex = Data$sex,
  male = "Male",
  female = "Female",
  ref = cdc.ref,
  item = "bmi",
  type = "perc"
)

```


Linear regression for the relationship between interaction of Picky eating and parental feeding styles on Child BMI and SNAP/WIC

```{r}

#Linear regression for the relationship between interaction of Picky eating and Control feeding on Child BMI and SNAP/WIC
lm <- lm(BMIpercentile ~ Picky_Score * Control_Score + WIC_or_SNAP + Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+ security, data = Data)
summary(lm)

lm222 <- lm(BMIpercentile ~ Picky_Score * Control_Score * WIC_or_SNAP + Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security, data = Data)
summary(lm222)


#correlation matrix of predictors for the interaction of Picky eating,SNAP and Control feeding on Child BMI
correlation_matrix <- cor(Data[c("Picky_Score", "Control_Score", "WIC_or_SNAP", "Income", "Education", "Role", "Race", "household", "Relationship", "num_children", "Ethnicity", "security")], 
                          use = "pairwise.complete.obs")
 # View the correlation matrix
print(correlation_matrix)





#Linear regression for the relationship between BMI and interaction of Emotional eating, SNAP and Picky eating: 
lm2 <- lm(BMIpercentile~ Picky_Score * Emotional_Score + WIC_or_SNAP +Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security , data = Data)
summary(lm2)

lm22 <- glm(BMIpercentile~ Picky_Score * Emotional_Score *WIC_or_SNAP +Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security , data = Data)
summary(lm22)



#correlation matrix of predictors for the relationship between BMI and interaction of Emotional eating, SNAP and Picky eating:
correlation_matrix <- cor(Data[c("Picky_Score", "Emotional_Score", "WIC_or_SNAP", "Income", "Education", "Role", "Race", "household", "Relationship", "num_children", "Ethnicity", "security")], 
                          use = "pairwise.complete.obs")
 # View the correlation matrix
print(correlation_matrix)






#GLM model for the relationship between BMI and interaction of encouragement eating, SNAP and Picky eating:

lm3 <- lm(BMIpercentile~ Picky_Score * Encourage_Score + WIC_or_SNAP+ Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security , data = Data)
summary(lm3)
lm3333 <- lm(BMIpercentile~ Picky_Score * Encourage_Score * WIC_or_SNAP + Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security, data = Data)
summary(lm3333)


#correlation matrix for the relationship between BMI and interaction of encouragement eating, SNAP and Picky eating:
correlation_matrix <- cor(Data[c("Picky_Score", "Encourage_Score", "WIC_or_SNAP", "Income", "Education", "Role", "Race", "household", "Relationship", "num_children", "Ethnicity", "security")], 
                          use = "pairwise.complete.obs")
 # View the correlation matrix
print(correlation_matrix)






#GLM for the relationship between BMI and interaction of instrumental feeding style , SNAP and Picky eating on BMI

lm4 <- lm(BMIpercentile~ Picky_Score * Instru_Score + WIC_or_SNAP + Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security , data = Data)
summary(lm4)

lm444 <- lm(BMIpercentile~ Picky_Score* Instru_Score * WIC_or_SNAP +Income + Education + Role + household+ Relationship +num_children+Race+Ethnicity+security, data = Data)
summary(lm444)





#correlation matrix of predictors for the relationship between BMI and interaction of instrumental feeding style , SNAP and Picky eating on BMI
correlation_matrix <- cor(Data[c("Picky_Score", "Instrue_Score", "WIC_or_SNAP", "Income", "Education", "Role", "Race", "household", "Relationship", "num_children", "Ethnicity", "security")], 
                          use = "pairwise.complete.obs")
 # View the correlation matrix
print(correlation_matrix)

```




Diagram for the relationship between interaction of Picky eating, SNAP and Parental feeding styls on Child BMI 

```{r}
#Diagram for the relationship between BMI and interaction of controlling feeding style, SNAP and Picky eating:
# Plot BMI vs. Emotional Score with categorized Picky Eating Score and separate strict regression lines
ggplot(Data, aes(x = Control_Score, y = BMIpercentile, color = Picky_Category)) +
  geom_point() + 
  geom_smooth(aes(group = Picky_Category, color = Picky_Category), 
              method = "lm", formula = y ~ x, se = TRUE, linetype = "solid") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE, linetype = "solid", color = "black") + # Overall regression line
  labs(title = "BMI Percentile vs. Control Feeding Style",
       x = "Control Feeding Style", 
       y = "BMI Percentile",
       color = "Picky Eating ") +
  scale_color_manual(values = c("Low" = "blue", "Medium" = "green", "High" = "red"),
                     name = "Picky Eating") +
  annotate("text", x = max(Data$Control_Score), y = min(Data$BMIpercentile), 
           label = "Overall Effect of Picky Eating", hjust = 1, vjust = 0) +
  theme_minimal()
ggsave("g1.png", dpi = 300)




#Diagram for the relationship between BMI and interaction of Emotional feeding style, SNAP and Picky eating:
# Plot BMI vs. Emotional Score with categorized Picky Eating Score and separate strict regression lines
ggplot(Data, aes(x = Emotional_Score, y = BMIpercentile, color = Picky_Category)) +
  geom_point() + 
  geom_smooth(aes(group = Picky_Category, color = Picky_Category), 
              method = "lm", formula = y ~ x, se = TRUE, linetype = "solid") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE, linetype = "solid", color = "black") + # Overall regression line
  labs(title = "BMI Percentile vs. Emotional Feeding Style",
       x = "Emotional Feeding Style", 
       y = "BMI Percentile",
       color = "Picky Eating") +
  scale_color_manual(values = c("Low" = "blue", "Medium" = "green", "High" = "red"),
                     name = "Picky Eating") +
  annotate("text", x = max(Data$Emotional_Score), y = min(Data$BMIpercentile), 
           label = "Overall Effect of Picky Eating", hjust = 1, vjust = 0) +
  theme_minimal()

ggsave("g2.png", dpi = 300)






#Diagram for the relationship between BMI and interaction of encouragement feeding style, SNAP and Picky eating:
# Plot BMI vs. Emotional Score with categorized Picky Eating Score and separate strict regression lines
ggplot(Data, aes(x = Encourage_Score, y = BMIpercentile, color = Picky_Category)) +
  geom_point() + 
  geom_smooth(aes(group = Picky_Category, color = Picky_Category), 
              method = "lm", formula = y ~ x, se = TRUE, linetype = "solid") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE, linetype = "solid", color = "black") + # Overall regression line
  labs(title = "BMI Percentile vs. Encouragemental Feeding Style",
       x = "Encouragemental Feeding Style", 
       y = "BMI Percentile",
       color = "Picky Eating") +
  scale_color_manual(values = c("Low" = "blue", "Medium" = "green", "High" = "red"),
                     name = "Picky Eating") +
  annotate("text", x = max(Data$Encourage_Score), y = min(Data$BMIpercentile), 
           label = "Overall Effect of Picky Eating", hjust = 1, vjust = 0) +
  theme_minimal()
ggsave("g3.png", dpi = 300)




#Diagram for the relationship between BMI and interaction of instrumental feeding style , SNAP and Picky eating on BMI
# Plot BMI vs. Emotional Score with categorized Picky Eating Score and separate strict regression lines
ggplot(Data, aes(x = Instru_Score, y = BMIpercentile, color = Picky_Category)) +
  geom_point() + 
  geom_smooth(aes(group = Picky_Category, color = Picky_Category), 
              method = "lm", formula = y ~ x, se = TRUE, linetype = "solid") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE, linetype = "solid", color = "black") + # Overall regression line
  labs(title = "BMI Percentile vs. Instrumental Feeding Style",
       x = "Instrumental Feeding Style", 
       y = "BMI Percentile",
       color = "Picky Eating") +
  scale_color_manual(values = c("Low" = "blue", "Medium" = "green", "High" = "red"),
                     name = "Picky Eating") +
  annotate("text", x = max(Data$Instru_Score), y = min(Data$BMIpercentile), 
           label = "Overall Effect of Picky Eating", hjust = 1, vjust = 0) +
  theme_minimal()

ggsave("g4.png", dpi = 300)
```





Demographic table for sample size 173



```{r}
#Mean and SD Table
ECfinal %>%
  summarise(
    Agemean = mean(Age, na.rm= TRUE),
    Agesd = sd(Age, na.rm = TRUE),
    securitymean = mean(security, na.rm= TRUE),
    securitysd = sd(security, na.rm= TRUE),
    EncouragementallFeedingStyleMean = mean(Encourage_Score, na.rm = TRUE),
    EncouragementallFeedingStyleSD = sd(Encourage_Score, na.rm = TRUE),
    InstrumentalFeedingStyleMean = mean(Instru_Score, na.rm = TRUE),
    InstrumentalFeedingStyleSD = sd(Instru_Score, na.rm = TRUE),
    EmotionalFeedingStyleMean = mean(Emotional_Score, na.rm = TRUE),
    EmotionalFeedingStyleSD = sd(Emotional_Score, na.rm = TRUE),
    ControlFeedingStyleMean = mean(Control_Score, na.rm = TRUE),
    ControlFeedingStyleSD = sd(Control_Score, na.rm = TRUE),
    PickyEatingMean = mean(Picky_Score, na.rm = TRUE),
    PickyEatingSD = sd(Picky_Score, na.rm = TRUE),
  )
# Group by SNAP
ECfinal %>%
  group_by(WIC_or_SNAP) %>%
  summarise(
    Agemean = mean(Age, na.rm= TRUE),
    Agesd = sd(Age, na.rm = TRUE),
    securitymean = mean(security, na.rm= TRUE),
    securitysd = sd(security, na.rm= TRUE),
    EncouragementallFeedingStyleMean = mean(Encourage_Score, na.rm = TRUE),
    EncouragementallFeedingStyleSD = sd(Encourage_Score, na.rm = TRUE),
    InstrumentalFeedingStyleMean = mean(Instru_Score, na.rm = TRUE),
    InstrumentalFeedingStyleSD = sd(Instru_Score, na.rm = TRUE),
    EmotionalFeedingStyleMean = mean(Emotional_Score, na.rm = TRUE),
    EmotionalFeedingStyleSD = sd(Emotional_Score, na.rm = TRUE),
    ControlFeedingStyleMean = mean(Control_Score, na.rm = TRUE),
    ControlFeedingStyleSD = sd(Control_Score, na.rm = TRUE),
    PickyEatingMean = mean(Picky_Score, na.rm = TRUE),
    PickyEatingSD = sd(Picky_Score, na.rm = TRUE)
  )








#Demographic Table percentage totally
# Function to calculate the count and percentage of each option in a categorical variable, excluding NA values
calculate_distribution <- function(data, column_name) {
  total_non_na <- sum(!is.na(data[[column_name]]))  # Total count of non-NA values for the variable
  data %>%
    filter(!is.na(!!sym(column_name))) %>%  # Exclude NA values
    group_by(!!sym(column_name)) %>%
    summarise(
      Count = n(),
      Percentage =round((Count / total_non_na) * 100,2),  # Calculate percentage based on total non-NA count
      .groups = 'drop'
    )
}

# Apply the function to each variable and print the results
print(calculate_distribution(ECfinal, "Education"))
print(calculate_distribution(ECfinal, "Income"))
print(calculate_distribution(ECfinal, "Role"))
print(calculate_distribution(ECfinal, "WIC_or_SNAP"))
ECfinal$household[ECfinal$household == 1] <- 2
print(calculate_distribution(ECfinal, "household"))
print(calculate_distribution(ECfinal, "num_children"))
print(calculate_distribution(ECfinal, "Sex"))
print(calculate_distribution(ECfinal, "Relationship"))
print(calculate_distribution(ECfinal, "Race"))
print(calculate_distribution(ECfinal, "Ethnicity"))

# Function to calculate the count and percentage of each option in a categorical variable within the SNAP and non-SNAP groups
calculate_distribution_snap_non_snap <- function(data, column_name) {
  snap_data <- data %>%
    filter(WIC_or_SNAP == 1 & !is.na(!!sym(column_name)))  # Filter for SNAP group and remove NAs
  non_snap_data <- data %>%
    filter(WIC_or_SNAP == 0 & !is.na(!!sym(column_name)))  # Filter for non-SNAP group and remove NAs
  
  total_snap <- nrow(snap_data)  # Total count within the SNAP group
  total_non_snap <- nrow(non_snap_data)  # Total count within the non-SNAP group
  
  snap_result <- snap_data %>%
    group_by(!!sym(column_name)) %>%
    summarise(
      Count = n(),
      Percentage = round((Count / total_snap) * 100, 2),  # Calculate percentage based on count within SNAP
      .groups = 'drop'
    )
  
  non_snap_result <- non_snap_data %>%
    group_by(!!sym(column_name)) %>%
    summarise(
      Count = n(),
      Percentage = round((Count / total_non_snap) * 100, 2),  # Calculate percentage based on count within non-SNAP
      .groups = 'drop'
    )
  
  list(SNAP = snap_result, Non_SNAP = non_snap_result)  # Return results for both groups
}

# Calculate distribution for Income variable within SNAP and non-SNAP groups
income_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Income")
print(income_distribution$SNAP)
print(income_distribution$Non_SNAP) 

# Calculate distribution for parents role
gender_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Role")
print(gender_distribution$SNAP)
print(gender_distribution$Non_SNAP)

# Calculate distribution for Household 
household_distribution<- calculate_distribution_snap_non_snap(ECfinal, "household")
print(household_distribution$SNAP)
print(household_distribution$Non_SNAP)

# Calculate distribution for Number of Children
num_children_distribution <- calculate_distribution_snap_non_snap(ECfinal, "num_children")
print(num_children_distribution$SNAP)
print(num_children_distribution$Non_SNAP)

# Calculate distribution for Sex 
sex_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Sex")
print(sex_distribution$SNAP)  
print(sex_distribution$Non_SNAP)

# Calculate distribution for Relationship
relationship_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Relationship")
print(relationship_distribution$SNAP)
print(relationship_distribution$Non_SNAP) 

# Calculate distribution for Race 
race_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Race")
print(race_distribution$SNAP)
print(race_distribution$Non_SNAP)
# Calculate distribution for Ethnicity 
ethnicity_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Ethnicity")
print(ethnicity_distribution$SNAP) 
print(ethnicity_distribution$Non_SNAP)

# Calculate distribution for Education variable 
education_distribution <- calculate_distribution_snap_non_snap(ECfinal, "Education")
print(education_distribution$SNAP)  
print(education_distribution$Non_SNAP)  


```

Demographic pie chart173

```{r}


# Function to generate color palette
generate_color_palette <- function(data, category_name) {
  categories <- unique(data[[category_name]])
  colors <- hcl.colors(length(categories), "Set3")
  names(colors) <- categories
  return(colors)
}

# Adjusted plotting function
plot_distribution <- function(snap_data, non_snap_data, category_name) {
  # Prepare data
  snap_data$Type <- "SNAP"
  non_snap_data$Type <- "Non-SNAP"
  combined_data <- rbind(snap_data, non_snap_data)
  combined_data$Category <- as.factor(combined_data[[category_name]]) # Dynamically use category name
  
  # Generate color palette based on combined data
  colors <- generate_color_palette(combined_data, category_name)
  
  # Calculate midpoint angles for labels
  combined_data <- combined_data %>%
    arrange(Type, desc(Category)) %>%
    group_by(Type) %>%
    mutate(
      cumulative_percentage = cumsum(Percentage),
      midpoint = cumulative_percentage - (.5 * Percentage),
      label_position = 1.1  # Adjust this value to move labels outside the pie
    )
  
  p <- ggplot(combined_data, aes(x = 2, y = Percentage, fill = Category)) +
    geom_bar(stat = "identity", color = "white", width = 1) +
    coord_polar("y", start = 0) +
    scale_fill_manual(values = colors) +
    facet_wrap(~Type, ncol = 1) +
    theme_void() +
    theme(legend.position = "left")
  
  # Add labels
  p + geom_text(data = combined_data, aes(x = 2.1, y =midpoint, label = paste( paste0(round(Percentage, 1), "%"), sep = "  : ")),
                position = position_nudge(y = 0.3), hjust = .9, color = "black", size = 3.5, angle = 90)
}




# Calls to plot_distribution for each variable
plot_distribution(income_distribution$SNAP, income_distribution$Non_SNAP, "Income")
plot_distribution(gender_distribution$SNAP, gender_distribution$Non_SNAP, "Role")
plot_distribution(household_distribution$SNAP, household_distribution$Non_SNAP, "household")
plot_distribution(num_children_distribution$SNAP, num_children_distribution$Non_SNAP, "num_children")
plot_distribution(sex_distribution$SNAP, sex_distribution$Non_SNAP, "Sex")
plot_distribution(relationship_distribution$SNAP, relationship_distribution$Non_SNAP, "Relationship")
plot_distribution(race_distribution$SNAP, race_distribution$Non_SNAP, "Race")
plot_distribution(ethnicity_distribution$SNAP, ethnicity_distribution$Non_SNAP, "Ethnicity")
plot_distribution(education_distribution$SNAP, education_distribution$Non_SNAP, "Education")




#save pie charts
save_plot <- function(plot, filename) {
  ggsave(plot = plot, filename = filename, width = 8, height = 6, dpi = 300)
}

plot_income <- plot_distribution(income_distribution$SNAP, income_distribution$Non_SNAP, "Income")
save_plot(plot_income, "Income_Distribution.png")
plot_gender <- plot_distribution(gender_distribution$SNAP, gender_distribution$Non_SNAP, "Role")
save_plot(plot_gender, "Gender_Distribution.png")
plot_household <- plot_distribution(household_distribution$SNAP, household_distribution$Non_SNAP, "household")
save_plot(plot_household, "Household_Distribution.png")
plot_num_children <- plot_distribution(num_children_distribution$SNAP, num_children_distribution$Non_SNAP, "num_children")
save_plot(plot_num_children, "Num_Children_Distribution.png")
plot_sex <- plot_distribution(sex_distribution$SNAP, sex_distribution$Non_SNAP, "Sex")
save_plot(plot_sex, "Sex_Distribution.png")
plot_relationship <- plot_distribution(relationship_distribution$SNAP, relationship_distribution$Non_SNAP, "Relationship")
save_plot(plot_relationship, "Relationship_Distribution.png")
plot_race <- plot_distribution(race_distribution$SNAP, race_distribution$Non_SNAP, "Race")
save_plot(plot_race, "Race_Distribution.png")
plot_ethnicity <- plot_distribution(ethnicity_distribution$SNAP, ethnicity_distribution$Non_SNAP, "Ethnicity")
save_plot(plot_ethnicity, "Ethnicity_Distribution.png")
plot_education <- plot_distribution(education_distribution$SNAP, education_distribution$Non_SNAP, "Education")
save_plot(plot_education, "Education_Distribution.png")

# Confirm completion
print("All plots have been generated and saved.")


```




 Demographic table for sample size 94



```{r}




# mean and sd Table
Data %>%
  summarise(
    securitymean = mean(security, na.rm= TRUE),
    securitysd = sd(security, na.rm= TRUE),
    BMIPercentileMean = 100*mean(BMIpercentile, na.rm = TRUE),
    BMIPercentileSD = sd(BMIpercentile, na.rm = TRUE),
    BMImean = mean(BMI, na.rm = TRUE),
    BMIsd = sd(BMI, na.rm = TRUE),
    Agemean = mean(Age, na.rm= TRUE),
    Agesd = sd(Age, na.rm = TRUE),
    EncouragementallFeedingStyleMean = mean(Encourage_Score, na.rm = TRUE),
    EncouragementallFeedingStyleSD = sd(Encourage_Score, na.rm = TRUE),
    InstrumentalFeedingStyleMean = mean(Instru_Score, na.rm = TRUE),
    InstrumentalFeedingStyleSD = sd(Instru_Score, na.rm = TRUE),
    EmotionalFeedingStyleMean = mean(Emotional_Score, na.rm = TRUE),
    EmotionalFeedingStyleSD = sd(Emotional_Score, na.rm = TRUE),
    ControlFeedingStyleMean = mean(Control_Score, na.rm = TRUE),
    ControlFeedingStyleSD = sd(Control_Score, na.rm = TRUE),
    PickyEatingMean = mean(Picky_Score, na.rm = TRUE),
    PickyEatingSD = sd(Picky_Score, na.rm = TRUE),
  )

# Group by SNAP
Data %>%
  group_by(WIC_or_SNAP) %>%
  summarise(
    securitymean = mean(security, na.rm= TRUE),
    securitysd = sd(security, na.rm= TRUE),
    BMIPercentileMean = 100*mean(BMIpercentile, na.rm = TRUE),
    BMIPercentileSD = sd(BMIpercentile, na.rm = TRUE),
    BMImean = mean(BMI, na.rm = TRUE),
    BMIsd = sd(BMI, na.rm = TRUE),
    Agemean = mean(Age, na.rm= TRUE),
    Agesd = sd(Age, na.rm = TRUE),
    EncouragementallFeedingStyleMean = mean(Encourage_Score, na.rm = TRUE),
    EncouragementallFeedingStyleSD = sd(Encourage_Score, na.rm = TRUE),
    InstrumentalFeedingStyleMean = mean(Instru_Score, na.rm = TRUE),
    InstrumentalFeedingStyleSD = sd(Instru_Score, na.rm = TRUE),
    EmotionalFeedingStyleMean = mean(Emotional_Score, na.rm = TRUE),
    EmotionalFeedingStyleSD = sd(Emotional_Score, na.rm = TRUE),
    ControlFeedingStyleMean = mean(Control_Score, na.rm = TRUE),
    ControlFeedingStyleSD = sd(Control_Score, na.rm = TRUE),
    PickyEatingMean = mean(Picky_Score, na.rm = TRUE),
    PickyEatingSD = sd(Picky_Score, na.rm = TRUE)
  )



#Demographic percent for sample size 94 
# Function to calculate the count and percentage of each option in a categorical variable, excluding NA values
calculate_distribution <- function(data, column_name) {
  total_non_na <- sum(!is.na(data[[column_name]]))  # Total count of non-NA values for the variable
  data %>%
    filter(!is.na(!!sym(column_name))) %>%  # Exclude NA values
    group_by(!!sym(column_name)) %>%
    summarise(
      Count = n(),
      Percentage = (Count / total_non_na) * 100,  # Calculate percentage based on total non-NA count
      .groups = 'drop'
    )
}
Data <- Data%>% 
  filter(household >= 2)
# Apply the function to each variable and print the results
print(calculate_distribution(Data, "Education"))
print(calculate_distribution(Data, "Income"))
print(calculate_distribution(Data, "Role"))
print(calculate_distribution(Data, "WIC_or_SNAP"))
print(calculate_distribution(Data, "household"))
print(calculate_distribution(Data, "num_children"))
print(calculate_distribution(Data, "Sex"))
print(calculate_distribution(Data, "Relationship"))
print(calculate_distribution(Data, "Race"))
print(calculate_distribution(Data, "Ethnicity"))



# Function to calculate the count and percentage of each option in a categorical variable within the SNAP and non-SNAP groups
calculate_distribution_snap_non_snap <- function(data, column_name) {
  snap_data <- data %>%
    filter(WIC_or_SNAP == 1 & !is.na(!!sym(column_name)))  # Filter for SNAP group and remove NAs
  non_snap_data <- data %>%
    filter(WIC_or_SNAP == 0 & !is.na(!!sym(column_name)))  # Filter for non-SNAP group and remove NAs
  
  total_snap <- nrow(snap_data)  # Total count within the SNAP group
  total_non_snap <- nrow(non_snap_data)  # Total count within the non-SNAP group
  
  snap_result <- snap_data %>%
    group_by(!!sym(column_name)) %>%
    summarise(
      Count = n(),
      Percentage = round((Count / total_snap) * 100, 2),  # Calculate percentage based on count within SNAP
      .groups = 'drop'
    )
  
  non_snap_result <- non_snap_data %>%
    group_by(!!sym(column_name)) %>%
    summarise(
      Count = n(),
      Percentage = round((Count / total_non_snap) * 100, 2),  # Calculate percentage based on count within non-SNAP
      .groups = 'drop'
    )
  
  list(SNAP = snap_result, Non_SNAP = non_snap_result)  # Return results for both groups
}

# Calculate distribution for Income variable within SNAP and non-SNAP groups
income_distribution1 <- calculate_distribution_snap_non_snap(Data, "Income")
print(income_distribution1$SNAP)
print(income_distribution1$Non_SNAP) 

# Calculate distribution for Gender 
gender_distribution1 <- calculate_distribution_snap_non_snap(Data, "Role")
print(gender_distribution1$SNAP)
print(gender_distribution1$Non_SNAP)

# Calculate distribution for Household 
household_distribution1 <- calculate_distribution_snap_non_snap(Data, "household")
print(household_distribution1$SNAP)
print(household_distribution1$Non_SNAP)

# Calculate distribution for Number of Children
num_children_distribution1 <- calculate_distribution_snap_non_snap(Data, "num_children")
print(num_children_distribution1$SNAP)
print(num_children_distribution1$Non_SNAP)

# Calculate distribution for Sex 
sex_distribution1 <- calculate_distribution_snap_non_snap(Data, "Sex")
print(sex_distribution1$SNAP)  
print(sex_distribution1$Non_SNAP)

# Calculate distribution for Relationship
relationship_distribution1 <- calculate_distribution_snap_non_snap(Data, "Relationship")
print(relationship_distribution1$SNAP)
print(relationship_distribution1$Non_SNAP) 

# Calculate distribution for Race 
race_distribution1 <- calculate_distribution_snap_non_snap(Data, "Race")
print(race_distribution1$SNAP)
print(race_distribution1$Non_SNAP)
# Calculate distribution for Ethnicity 
ethnicity_distribution1 <- calculate_distribution_snap_non_snap(Data, "Ethnicity")
print(ethnicity_distribution1$SNAP) 
print(ethnicity_distribution1$Non_SNAP)

# Calculate distribution for Education variable 
education_distribution1 <- calculate_distribution_snap_non_snap(Data, "Education")
print(education_distribution1$SNAP)  
print(education_distribution1$Non_SNAP)  

```

Demographic pi chart

```{r}
# Function to generate color palette
generate_color_palette <- function(data, category_name) {
  categories <- unique(data[[category_name]])
  colors <- hcl.colors(length(categories), "Set3")
  names(colors) <- categories
  return(colors)
}

# Adjusted plotting function
plot_distribution <- function(snap_data, non_snap_data, category_name) {
  # Prepare data
  snap_data$Type <- "SNAP"
  non_snap_data$Type <- "Non-SNAP"
  combined_data <- rbind(snap_data, non_snap_data)
  combined_data$Category <- as.factor(combined_data[[category_name]]) # Dynamically use category name
  
  # Generate color palette based on combined data
  colors <- generate_color_palette(combined_data, category_name)
  
  # Calculate midpoint angles for labels
  combined_data <- combined_data %>%
    arrange(Type, desc(Category)) %>%
    group_by(Type) %>%
    mutate(
      cumulative_percentage = cumsum(Percentage),
      midpoint = cumulative_percentage - (.5 * Percentage),
      label_position = 1.1  # Adjust this value to move labels outside the pie
    )
  
  p <- ggplot(combined_data, aes(x = 2, y = Percentage, fill = Category)) +
    geom_bar(stat = "identity", color = "white", width = 1) +
    coord_polar("y", start = 0) +
    scale_fill_manual(values = colors) +
    facet_wrap(~Type, ncol = 1) +
    theme_void() +
    theme(legend.position = "left")
  
  # Add labels
  p + geom_text(data = combined_data, aes(x = 2.3, y =midpoint, label = paste( paste0(round(Percentage, 1), "%"), sep = "  : ")),
                position = position_nudge(y = 0.3), hjust = .5, color = "black", size = 2.5, angle = 90)
}




# Calls to plot_distribution for each variable
plot_distribution(income_distribution1$SNAP, income_distribution1$Non_SNAP, "Income")
plot_distribution(gender_distribution1$SNAP, gender_distribution1$Non_SNAP, "Role")
plot_distribution(household_distribution1$SNAP, household_distribution1$Non_SNAP, "household")
plot_distribution(num_children_distribution1$SNAP, num_children_distribution1$Non_SNAP, "num_children")
plot_distribution(sex_distribution1$SNAP, sex_distribution1$Non_SNAP, "Sex")
plot_distribution(relationship_distribution1$SNAP, relationship_distribution1$Non_SNAP, "Relationship")
plot_distribution(race_distribution1$SNAP, race_distribution1$Non_SNAP, "Race")
plot_distribution(ethnicity_distribution1$SNAP, ethnicity_distribution1$Non_SNAP, "Ethnicity")
plot_distribution(education_distribution1$SNAP, education_distribution1$Non_SNAP, "Education")




save_plot1 <- function(plot, filename) {
  ggsave(plot = plot, filename = filename, width = 8, height = 6, dpi = 300)
}

plot_income1 <- plot_distribution(income_distribution1$SNAP, income_distribution1$Non_SNAP, "Income")
save_plot(plot_income1, "Income_Distribution1.png")
plot_gender1 <- plot_distribution(gender_distribution1$SNAP, gender_distribution1$Non_SNAP, "Role")
save_plot(plot_gender1, "Gender_Distribution1.png")
plot_household1 <- plot_distribution(household_distribution1$SNAP, household_distribution1$Non_SNAP, "household")
save_plot(plot_household1, "Household_Distribution11.png")
plot_num_children1 <- plot_distribution(num_children_distribution1$SNAP, num_children_distribution1$Non_SNAP, "num_children")
save_plot(plot_num_children1, "Num_Children_Distribution1.png")
plot_sex <- plot_distribution(sex_distribution1$SNAP, sex_distribution1$Non_SNAP, "Sex")
save_plot(plot_sex1, "Sex_Distribution1.png")
plot_relationship1 <- plot_distribution(relationship_distribution1$SNAP, relationship_distribution1$Non_SNAP, "Relationship")
save_plot(plot_relationship1, "Relationship_Distribution1.png")
plot_race <- plot_distribution(race_distribution1$SNAP, race_distribution1$Non_SNAP, "Race")
save_plot(plot_race1, "Race_Distribution1.png")
plot_ethnicity <- plot_distribution(ethnicity_distribution1$SNAP, ethnicity_distribution1$Non_SNAP, "Ethnicity")
save_plot(plot_ethnicity1, "Ethnicity_Distribution1.png")
plot_education <- plot_distribution(education_distribution1$SNAP, education_distribution1$Non_SNAP, "Education")
save_plot(plot_education1, "Education_Distribution1.png")

# Confirm completion
print("All plots have been generated and saved.")



```






