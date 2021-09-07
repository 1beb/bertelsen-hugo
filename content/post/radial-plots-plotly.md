---
author: "Brandon Erik Bertelsen"
title: "Radial Plots with Plotly"
date: "2016-06-14"
categories:
- R
- Plotting
---

How to make a radial plot without using radial coordinates with plotly.

Let's load a few libraries and pull some sample data. Then do some preparation by adding data associated with turning the graphic into a circle. A trivial application of high school [SOHCATTOA](https://en.wikipedia.org/wiki/Trigonometry#Mnemonics), and finally plot. Comments included at each step.

```R
# Loading ---------------------------------
library(plotly)  
library(dplyr) # just because!  
source("https://git.io/vole5") # load data

# Data Preperation ------------------------
df$theta <- seq(0,345,15) # 24 responses, equals 15 degrees per response  
df$o <- df$Proportion * sin(df$theta * pi / 180) # SOH  
df$a <- df$Proportion * cos(df$theta * pi / 180) # CAH  
df$o100 <- 1 * sin(df$theta * pi / 180) # Outer ring x  
df$a100 <- 1 * cos(df$theta * pi / 180) # Outer ring y  
df$a75 <- 0.75 * cos(df$theta * pi / 180) # 75% ring y  
df$o75 <- 0.75 * sin(df$theta * pi / 180) # 75% ring x  
df$o50 <- 0.5 * sin(df$theta * pi / 180) # 50% ring x  
df$a50 <- 0.5 * cos(df$theta * pi / 180) # 50% ring y

# Plot ------------------------------------
p <- plot_ly()

# Wire-frame lines from origin
for(i in 1:24) {  
  p <- add_trace(
    p, 
    x = c(d$o100[i],0), 
    y = c(d$a100[i],0), 
    evaluate = TRUE,
    line = list(color = "#d3d3d3", dash = "3px"),
    showlegend = FALSE
    )
}

p %>%  
  # Add lines 
  add_trace(data = d[c(1:48,1,25),], x = o, y = a, color = Year, 
            mode = "lines+markers",
            hoverinfo = "text", 
            text = paste(Year, Response,round(Proportion * 100), "%")) %>%
  # Add 100% circle 
  add_trace(data = d, x = o100, y = a100, 
            text = Response,
            hoverinfo = "none",
            textposition = "top middle", mode = "lines+text", 
            line = list(color = "#d3d3d3", dash = "3px", shape = "spline"),
            showlegend = FALSE) %>% 
  # Add 50% circle
  add_trace(data = d, x = o50, y = a50, mode = "lines", 
            line = list(color = "#d3d3d3", dash = "3px", shape = "spline"), 
            hoverinfo = "none",
            showlegend = FALSE) %>%
  # Add 75% circle 
  add_trace(data = d, x = o75, y = a75, mode = "lines", 
            line = list(color = "#d3d3d3", dash = "3px", shape = "spline"), 
            hoverinfo = "none",
            showlegend = FALSE) %>%
  layout(
    autosize = FALSE,
    hovermode = "closest",     
    autoscale = TRUE,
    width = 800,
    height = 800,
    xaxis = list(range = c(-1.25,1.25), 
                 showticklabels = FALSE, 
                 zeroline = FALSE, showgrid = FALSE),
    yaxis = list(range = c(-1.25,1.25), 
                 showticklabels = FALSE, 
                 zeroline = FALSE, showgrid = FALSE)
  )
```