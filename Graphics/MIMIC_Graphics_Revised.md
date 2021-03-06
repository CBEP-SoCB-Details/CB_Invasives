Graphics for MIMIC Invasive Species Data
================
Curtis C. Bohlen, Casco Bay Estuary Partnership
3/16/2021

-   [Introduction](#introduction)
-   [Load Libraries](#load-libraries)
-   [Load Data](#load-data)
    -   [Establish Folder Reference](#establish-folder-reference)
-   [Import Fully QA/QC’d Data](#import-fully-qaqcd-data)
-   [Convert to Factors for Display
    Order](#convert-to-factors-for-display-order)
-   [Add Order Factors](#add-order-factors)
-   [Recent Data Only](#recent-data-only)
-   [Summary Statistics By Site](#summary-statistics-by-site)
    -   [Create Long Version for Facet
        Plot](#create-long-version-for-facet-plot)
-   [Faceted Plots](#faceted-plots)
    -   [Create Facet Labels](#create-facet-labels)
    -   [Wide Facet](#wide-facet)

<img
    src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
    style="position:absolute;top:10px;right:50px;" />

# Introduction

This Notebook provides an updated graphics for the MIMIC invasive
species monitoring program from Casco Bay. The primary changes are to
the labels used for each site, and rescaling the graphics

The Marine Invader Monitoring and Information Collaborative (MIMIC) in
Casco Bay is a partnership between CBEP, the Wells National Estuarine
Research Reserve (Wells NERR), and the regional MIMIC program. The
Regional effort includes participants from several other New England
States.

Wells NERR trains community scientists to identify (currently) 23
species of invasives, including tunicates, bryozoans, algae and
crustaceans. Scientists visit sites monthly between May and October and
document abundance of these non-native species.

# Load Libraries

``` r
library(tidyverse)
#> Warning: package 'tidyverse' was built under R version 4.0.5
#> -- Attaching packages --------------------------------------- tidyverse 1.3.1 --
#> v ggplot2 3.3.3     v purrr   0.3.4
#> v tibble  3.1.2     v dplyr   1.0.6
#> v tidyr   1.1.3     v stringr 1.4.0
#> v readr   1.4.0     v forcats 0.5.1
#> Warning: package 'tidyr' was built under R version 4.0.5
#> Warning: package 'dplyr' was built under R version 4.0.5
#> Warning: package 'forcats' was built under R version 4.0.5
#> -- Conflicts ------------------------------------------ tidyverse_conflicts() --
#> x dplyr::filter() masks stats::filter()
#> x dplyr::lag()    masks stats::lag()
library(readxl)

library(VGAM)
#> Loading required package: stats4
#> Loading required package: splines
#> 
#> Attaching package: 'VGAM'
#> The following object is masked from 'package:tidyr':
#> 
#>     fill
#library(readr)

library(GGally)
#> Warning: package 'GGally' was built under R version 4.0.5
#> Registered S3 method overwritten by 'GGally':
#>   method from   
#>   +.gg   ggplot2
#library(zoo)
#library(lubridate)  # here, for the make_datetime() function

library(CBEPgraphics)
load_cbep_fonts()
theme_set(theme_cbep())
```

# Load Data

## Establish Folder Reference

``` r
sibfldnm <- 'Derived_Data'
parent   <- dirname(getwd())
sibling  <- file.path(parent,sibfldnm)
dir.create(file.path(getwd(), 'figures'), showWarnings = FALSE)
```

# Import Fully QA/QC’d Data

``` r
fn <- 'Abundance_Data.csv'
abundance_data <- read_csv(file.path(sibling, fn),
                           col_types = cols(
                             Date = col_datetime(format = ""),
                             Site = col_character(),
                             Type = col_character(),
                             City = col_character(),
                             State = col_character(),
                             Salinity = col_double(),
                             Temp = col_double(),
                             Month = col_character(),
                             Year = col_integer(),
                             Where = col_character(),
                             Species = col_character(),
                             Common = col_character(),
                             Abundance = col_character()
                           )) %>%
  mutate(Type  = factor(Type, levels = c('Dock', 'Tidepool')),
         Month = factor(Month, levels = month.abb),
         Abundance = ordered(Abundance, levels = c('Absent', 'Rare', 'Few', 
                                                   'Common', 'Abundant')))
#> Warning: The following named parsers don't match the column names: State

fn <- 'Presence_Data.csv'
presence_data <- read_csv(file.path(sibling, fn),
                          col_types = cols(
                             Date = col_datetime(format = ""),
                             Site = col_character(),
                             Type = col_character(),
                             City = col_character(),
                             State = col_character(),
                             Salinity = col_double(),
                             Temp = col_double(),
                             Month = col_character(),
                             Year = col_integer(),
                             Where = col_character(),
                             Species = col_character(),
                             Common = col_character(),
                             Present = col_logical()
                           )) %>%
  mutate(Type  = factor(Type, levels = c('Dock', 'Tidepool')),
         Month = factor(Month, levels = month.abb))
#> Warning: The following named parsers don't match the column names: State
```

# Convert to Factors for Display Order

``` r
abundance_data <- abundance_data %>%
 mutate(Site = factor(Site, levels = 
                         c(  "Spring Point Marina",
                             "SMCC Dock", 
                             "Siegel's Reef",
                             
                             "Peaks Dock",
                             "Peaks Tidepool",
                             
                             "Great Diamond Island Dock", 
                             "Great Diamond Island Tidepool",
                             
                             "Long Island Dock",
                             "Fowler's Tide Pool",
                             
                             "Chandlers Wharf Dock",
                             #"Chebeague Island Boat Yard",
                             "Chebeague Stone Pier", 
                             "Waldo Point"
                         )),
         Where = factor(Where, levels = c("Mainland", "Peaks","Great Diamond",
                                          "Long", "Chebeague") ))
```

``` r
presence_data <- presence_data %>%
  mutate(Site = factor(Site, levels = 
                         c(  "Spring Point Marina",
                             "SMCC Dock", 
                             "Siegel's Reef",
                             
                             "Peaks Dock",
                             "Peaks Tidepool",
                             
                             "Great Diamond Island Dock", 
                             "Great Diamond Island Tidepool",
                             
                             "Long Island Dock",
                             "Fowler's Tide Pool",
                             
                             "Chandlers Wharf Dock",
                             "Chebeague Stone Pier", 
                             "Waldo Point"
                         )),
         Where = factor(Where, levels = c("Mainland", "Peaks","Great Diamond",
                                          "Long", "Chebeague") ))
```

# Add Order Factors

We need to organize graphics by island in consistent structure. We will
use a bar chart, organized by Island and a common sequence within island
groups. To facilitate that, we need a factor that orders sites
consistently within island groups. While we are at it, we create
alternate labels for the plots.

``` r
orders <- tribble (
  ~Site,                            ~Order,      ~Label,
  "Spring Point Marina",               1,         "Spring Point Marina",  
  "SMCC Dock",                         2,         "SMCC Dock",
  "Siegel's Reef",                     3,         "Siegel's Reef",  
  
  "Peaks Dock",                        1,          "Peaks Dock",          
  "Peaks Tidepool",                    2,          "Peaks Tidepool", 
  
  "Great Diamond Island Dock",         1,          "Great Diamond Dock",    
  "Great Diamond Island Tidepool",     2,          "Great Diamond Tidepool",
  
  "Long Island Dock",                  1,          "Long Island Dock",  
  "Fowler's Tide Pool",                2,          "Fowler's Beach",   
  
  "Chandlers Wharf Dock",              1,          "Chandlers Wharf",  
  "Chebeague Stone Pier",              2,          "Stone Pier",   
  "Waldo Point" ,                      3,          "Waldo Point")
```

``` r
abundance_data <- abundance_data %>%
  left_join(orders, by = 'Site')

presence_data <- presence_data %>%
  left_join(orders, by = 'Site')
```

# Recent Data Only

``` r
recent_data <- presence_data %>% 
  filter(Year > 2015)
```

# Summary Statistics By Site

``` r
total_spp <- recent_data %>%
  group_by(Site, Where, Type, Order, Label, Species) %>%
  summarize(Spotted = any(Present),
            .groups = 'drop_last') %>%
  group_by(Site, Where, Type, Order, Label) %>%
  summarize(Tot_Species = sum(Spotted),
            .groups = 'drop')

avg_spp <- recent_data %>%
  group_by(Site, Date) %>%
  summarize(spp_present = sum(Present),
            .groups = 'drop') %>%
  group_by(Site) %>%
  summarize(Avg_Species = mean(spp_present),
            .groups = 'drop')

total_spp <- total_spp %>%
  left_join(avg_spp, by = 'Site')
rm(avg_spp)
```

## Create Long Version for Facet Plot

``` r
total_spp_long <- total_spp %>%
  pivot_longer(c(Tot_Species, Avg_Species),
               names_to = 'Parameter', 
               values_to = 'Value')
```

# Faceted Plots

## Create Facet Labels

``` r
mylabs <- c('Total Invasives', 'Average Invasives\nPer Visit')
names(mylabs) = c('Tot_Species', 'Avg_Species')

mylabs
#>                    Tot_Species                    Avg_Species 
#>              "Total Invasives" "Average Invasives\nPer Visit"
```

## Wide Facet

We rescaled this to fit more closely with the final page layout.

``` r
ggplot(total_spp_long, aes(x = Where, y = Value, group = Order, fill = Type)) +
  geom_col(position = 'dodge') +
  
  geom_text(aes(label = Label, y = 0.25), 
            position = position_dodge(0.9),
            angle = 90, 
            hjust = 0,
            vjust = 0.25,
            size = 1.5) +
  
  facet_wrap(~Parameter, labeller = labeller(Parameter = mylabs),  scales = 'fixed') +
  
  ylab('') +
  xlab('') +
  
  scale_fill_manual(values = cbep_colors2()[c(2,4)], name = '') +
  
  guides(fill = guide_legend(override.aes = list(size = 0.25))) +
  
  theme_cbep(base_size = 9) +
  theme(axis.text.x = element_text(angle = 45, 
                                   size = 6, 
                                   hjust = 1
                                   ),
        legend.position = c(0.125, 0.85),
        legend.title = element_text(size = 6), 
        legend.text = element_text(size = 6),
        legend.key.size = unit(.125, "in"),
        axis.ticks.length.x = unit(0, 'cm'))
```

<img src="MIMIC_Graphics_Revised_files/figure-gfm/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

``` r
ggsave('figures/wide_facet_invasives_by_site_revised.pdf', device = cairo_pdf, 
       width = 3, height = 3.5)
```
