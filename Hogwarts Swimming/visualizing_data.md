---
title: "Homework 7"
author: "Allan Lu"
date: "March 27, 2020"
output:
  html_document:
    keep_md: yes
    df_print: paged
    toc: yes
  html_notebook:
    df_print: paged
    toc: yes
---




The column values are as follows: 

`No_Event`   - Number of the event during the meet  
`Event`      - Event title  
`Swimmers`   - Swimmer name  
`School`     - School of swimmer  
`Location`   - Location of meet  
`Date`       - Date of meet  
`TIME`       - Final performance time ("MIN:SEC:MSEC") for presentation purposes  
`PL`         - Finish place. If a swimmer was disqualified (`DQ`), it would appear here  
`Against`    - Team swam against  
`House.name` - House affiliation of swimmer  
`Month`      - Month of meet  
`Spells`     - Average number of spells used by swimmer to increase their speed  
`Times`      - Numeric times (seconds) for analysis purposes


## Chart of Spells vs Swimming Speed
Create a point chart of the relationship between the number of spells (y-axis) used and the swimming speed (x-axis).


```r
dat <- "Hogwarts_swimming_file.csv"
df <- read_csv(dat)

# Manage the data 
swim_spells <- drop_na(df)

# Create the plot
ggplot(swim_spells, aes(as_datetime(Times), Spells)) +
  geom_point() +
  geom_smooth() +
  scale_x_datetime(date_breaks = "3 min", date_labels = "%M:%S") +
  labs(x = "Performance Time (min:sec)",
       y = "Average Number of Spells Used",
       title = "Number of Spells Used to Swimming Time",
       subtitle = "Are the best swimmers from physical training or spellwork?") +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(color = "black"))
```

![](visualizing_data_files/figure-html/speed versus spells by house-1.png)<!-- -->

```r
# Save plot1 
ggsave("plot1.png", plot = last_plot(), device = 'png', dpi = 400)

# Create plot2
ggplot(swim_spells, aes(as_datetime(Times), Spells)) +
  geom_point(aes(color = House.name)) +
  geom_smooth(aes(color = House.name), se = F) +
  scale_color_manual(values=c("firebrick", "gold", "royalblue", "darkgreen")) +
  scale_x_datetime(date_breaks = "3 min", date_labels = "%M:%S") +
  labs(x = "Performance Time (min:sec)",
       y = "Average Number of Spells Used",
       color = "Houses",
       title = "Number of Spells Used to Swimming Time",
       subtitle = "Which houses are relying too much on magic to win?") +
  theme(panel.background = element_rect(fill = "white"),
        axis.line = element_line(color = "black"))
```

![](visualizing_data_files/figure-html/speed versus spells by house-2.png)<!-- -->

```r
# Save plot2
ggsave("plot2.png", plot = last_plot(), device = 'png', dpi = 400)
```

The more spells used during a race, the lower the performance time. Based on the two figures, it is impossible to achieve those swim times (sub 1 min) without using an exorbitant number of spells. In the second figure, one can see that Slytherins were using the most spells during the races, while Gryffindors used the fewest. Hufflepuffs and Ravenclaws were in the middle with the same average usages.


## Facet Wrap of Swim Times

Plot the improvement of the swimming times by house for the 100 yard events (FREE, BACK, FLY, and BREAST) over the duration of the season.


```r
# Manage the data 
swim_100 <- df %>%
  drop_na %>%
  select(Event, Date, TIME, Swimmers, House = House.name, Times) %>%
  filter(grepl("100", Event))

# Create the plot
ggplot(swim_100, aes(Date, as_datetime(Times))) +
  geom_point(aes(color = House)) +
  geom_smooth(aes(color = House)) +
  facet_wrap(~Event, nrow = 2, ncol = 2) +
  scale_color_manual(values=c("firebrick", "gold", "royalblue", "darkgreen")) +
  scale_y_datetime(date_breaks = "1 min", date_labels = "%M:%S") +
  labs(y = "Performance Time (min:sec)",
       color = "Houses",
       title = "100 Yard Swim Event Times",
       subtitle = "Hogwarts House Performance",
       caption = "Data from Oct 2019 to Jan 2020") +
  theme(panel.background = element_rect(fill = "white", color = "black"),
        strip.background = element_rect(fill = "plum", color = "black"))
```

![](visualizing_data_files/figure-html/100 events improvements-1.png)<!-- -->

```r
# Save the plot 
ggsave("plot3.png", plot = last_plot(), device = 'png', dpi = 400) 
```

Based on the 4 graphs, the Hogwarts students had improved their performance times from October through January. For the 100-yard fly and free, Griffyndors improved the fastest. While for the 100-yard breast and back, Ravenclaws lowered their times the earliest.


## Fastest Swimmers

Who are the fastest swimmers in January for the 100-yard events?


```r
# Manage the data 
swim_100 <- df %>%
  drop_na %>%
  select(Event, Month, TIME, Swimmers, House = House.name, Times) %>%
  filter(grepl("100", Event) & Month == "January")

# Analysis 
swim_fast <- swim_100 %>%
  group_by(Event) %>%
  top_n(-1, Times) %>%
  # Make the table 
  select(-Month, -Times)

swim_fast
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Event"],"name":[1],"type":["chr"],"align":["left"]},{"label":["TIME"],"name":[2],"type":["S3: hms"],"align":["right"]},{"label":["Swimmers"],"name":[3],"type":["chr"],"align":["left"]},{"label":["House"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"100 FLY","2":"00:54:07","3":"Ron Weasley","4":"Gryffindor"},{"1":"100  BREAST","2":"01:00:02","3":"Victoria Frobisher","4":"Gryffindor"},{"1":"100  BREAST","2":"01:00:02","3":"Andrew Kirke","4":"Gryffindor"},{"1":"100 BACK","2":"00:55:06","3":"Ginny Weasley","4":"Gryffindor"},{"1":"100 FREE","2":"00:51:04","3":"Albus Dumbledore","4":"Gryffindor"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


## Disqualifications

Identify the swimmer who disqualified the most during the past season.   


```r
swim_dq <- df %>%
  select(Swimmers, PL) %>%
  filter(grepl("DQ", PL)) %>%
  group_by(Swimmers) %>%
  count(PL, name="DQs") %>%
  ungroup() %>%
  top_n(1, DQs)

sprintf("%s had the most disqualifications with %i.", swim_dq$Swimmers, swim_dq$DQs)
```

```
## [1] "Daphne Greengrass had the most disqualifications with 15."
```


## Facet Grids

Create a `facet_grid` chart of your choice using the Hogwarts data.


### Facet Grid of 100-Yard Swim Events


```r
df2 <- df %>%
  drop_na() %>%
  rename(House = House.name) %>%
  filter(grepl("100", Event))

ggplot(df2, aes(as_datetime(Times), color = House, fill = House)) +
  geom_density(position = "stack") + 
  facet_grid(Event ~ Against) +
  scale_fill_manual(values=c("firebrick", "yellow", "royalblue", "darkgreen")) +
  scale_color_manual(values=c("gold2", "black", "darkgoldenrod", "gray")) +
  scale_x_datetime(date_breaks = "1.5 min", date_labels = "%M:%S") +
  scale_y_continuous(breaks=c(0, 0.03, 0.06, 0.09)) + 
  labs(x = "Performance Time (min:sec)",
       y = "Density",
       title = "Hogwarts 100-Yard Event Swim Times",
       subtitle = "By Event and Opposing School",
       caption = "Density Plot of Swim Times") +
  theme(panel.background = element_rect(fill = "white", color = "black"),
        strip.background = element_rect(fill = "cornsilk", color = "black"),
        legend.direction = "horizontal",
        legend.position = "bottom")
```

![](visualizing_data_files/figure-html/extra credit part 2.1-1.png)<!-- -->

```r
ggsave("plot4.png", plot = last_plot(), device = 'png', dpi = 400) 
```

Above is a density plot of the Hogwarts swim team's performance times in the 100-yard events. This figure allows the reader to see how each house performed in each type of race against each of the opposing schools. For example, in the 100-yard breast against Castelobruxo, Hufflepuffs and Slytherins performed very well having a race time around 1 minute and 30 seconds, while Ravenclaws performed poorly in the 2:30-3:00 range. But in the 100-yard free against Ilvermorny, Ravenclaw performed the best. In some cases, like the 100-yard free against Durmstrang, the perfromances were spread across all times.

Using this figure, a swim coach can see which houses perform better in what type of events and enter the students in those races.


### Facet Grid of Freestyle Events


```r
df3 <- df %>%
  drop_na() %>%
  rename(House = House.name) %>%
  filter(!grepl("RELAY", Event) & grepl("FREE", Event)) %>%
  mutate(Event = factor(Event, levels=c("50 FREE", "100 FREE", "200 FREE", "500 FREE")))

ggplot(df3, aes(House, Spells, color = House, fill = House)) +
  geom_violin() +
  facet_grid(Event ~ Against, scale = "free") +
  scale_fill_manual(values=c("firebrick", "yellow", "royalblue", "darkgreen")) +
  scale_color_manual(values=c("gold2", "black", "darkgoldenrod", "gray")) +
  scale_x_discrete(labels = c("G", "H", "R", "S")) +
  labs(y = "Number of Spells",
       color = "Houses",
       fill = "Houses",
       title = "Hogwarts Swim Team's Spell Usage in Freestyle Events",
       subtitle = "By Distance and Opposing School",
       caption = "Violin Plot of Spell Usage at Swimming Events") +
  theme(panel.background = element_rect(fill = "white", color = "black"),
        strip.background = element_rect(fill = "cornsilk", color = "black"),
        legend.direction = "horizontal",
        legend.position = "bottom")
```

![](visualizing_data_files/figure-html/extra credit part 2.2-1.png)<!-- -->

```r
ggsave("plot5.png", plot = last_plot(), device = 'png', dpi = 400)
```

A violin plot looks like a combination of a box and density plot. It shows the frequencies and distributions of a variable. In the above figure, a facet grid allows the creation of multiple violin plots scaled by the opposing school and the distance of the race. Each violin plot is also separated into the four Hogwarts houses to see the number of spells each house used in that race. Overall, Slytherins were casting more spells than the other Houses, especially in the 500-yard race. In the 50-yard events, between 150 to 200 spells were casted on average across all houses, even Gryffindors were furiously casting spells to swim faster.