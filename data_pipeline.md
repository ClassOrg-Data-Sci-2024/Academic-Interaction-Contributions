Data-processing Pipeline
================
Jack Rechsteiner
03/15/2024

- [Data overview](#data-overview)
- [Data pipeline](#data-pipeline)
- [Session Info](#session-info)

``` r
##Set knitr options (show both code and output, show output w/o leading #)
knitr::opts_chunk$set(echo = TRUE, include = TRUE, comment=NA)

#load tidyverse
library("tidyverse")
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.4.4     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
#load xml2
library(xml2)
```

# Data overview

The data used in this project comes from the [Michigan Corpus of
Academic Spoken English](https://quod.lib.umich.edu/cgi/c/corpus/corpus)
(MICASE) which contains transcripts of academic speech events recorded
at the University of Michigan in the late 1990s and early 2000s. The
transcripts are available as XML files which contain metadata on the
speech event and the participants involved, as well as the entire
transcript. This metadata includes information on the type of speech
event, the primary discourse mode, the level of interactivity,the number
of participants and speakers, the English language proficiency of
speakers, the academic roles of participants, and the gender and age
group for each speaker. Specifically, this project works with the 75
interactions that are categorized within MICASE as “Highly interactive”
or “Mostly interactive”. These 75 transcripts represent roughly 6,360
minutes of recorded speech interactions.

# Data pipeline

The code presented here reads in the 75 XML files for this project and
then extracts data from the nested structure of the XML files, so that
it can be input into a tibble that will be used for analysis.

``` r
#Creating a list of file names to be iterated over read_xml() with map()
file_list <- list.files("Data", pattern="*.xml", full.names=TRUE)

#Using map() to create a list from reading in all the XML files
xml_list <- map(file_list, read_xml)

#Creating a list of all the file names from xml_list
##which are tagged as an "ID" attribute in the XML files
filename_list <- map(xml_list,
                     ~xml_attr(.x, "ID"))

#Creating a list of all the speechevent labels from xml_list
##by finding all the terms with 'speechevent' type and extracting the text
speechevent_list <- map(xml_list,
                        ~xml_find_all(.x, "//TERM[@TYPE='SPEECHEVENT']") %>% 
                          xml_text())

#Getting all the first-level utterance nodes from the BODY nodes in xml_list
speaker_list <- map(xml_list,
                    ~xml_find_all(.x, xpath="//BODY//U1"))

#Getting all the second-level speaker nodes as a separate element 
u2_speaker_list <- map(speaker_list,
                       ~xml_find_all(.x, xpath="//U1//U2"))

#Copying speaker_list because xml_remove() overwrites its input
u1_speaker_list <- speaker_list

#Using walk() to remove second-level speaker nodes from u1_speaker_list
##without printing the output of the function
walk(u1_speaker_list,
     ~ xml_remove(xml_find_all(.x, xpath="//U1//U2")))

#Turning speaker node attributes into chr lists for u1_speaker_list and u2_speaker_list
u1_speaker_nodes <- map(u1_speaker_list,
                        ~xml_attrs(.x))
u2_speaker_nodes <- map(u2_speaker_list,
                        ~xml_attrs(.x))

#Getting text content from speaker nodes for u1_speaker_list and u2_speaker_list
u1_speaker_text <- map(u1_speaker_list,
                       ~xml_text(.x))
u2_speaker_text <- map(u2_speaker_list,
                       ~xml_text(.x))

#Creating tibble with the columns:
##'file' using the filename string in filename_list
micase_df <- tibble(file = filename_list, 
                    ##'speechevent' using the speechevent string in speechevent_list
                    speechevent = speechevent_list, 
                    ##'level' to indicate whether it is a first-level or second-level utterance
                    level = "u1", 
                    ##'nodeset' which contains the lists of attributes for speakers
                    nodeset = u1_speaker_nodes, 
                    ##'text' which contains the text from the speaker
                    text = u1_speaker_text) %>% 
  #Using add_row() to add the data from the second-level utterances and set the level to "u2"
  add_row(file = filename_list, 
          speechevent = speechevent_list, 
          level = "u2", 
          nodeset = u2_speaker_nodes, 
          text = u2_speaker_text) %>% 
  #Using unnest() on the columns that contain lists created by map()
  unnest(cols = c(file, speechevent, nodeset, text)) %>% 
  #Giving each entry a unique_id based on row_number()
  mutate(unique_id = row_number(), .before=1) %>% 
  #Using unnest_wider() on the nodeset lists so that the XML attribute titles become columns 
  ##which are filled with the XML attribute values
  unnest_wider(nodeset)

#Brief samples of the current data frame
head(micase_df)
```

    # A tibble: 6 × 12
      unique_id file  speechevent level WHO   NSS   ROLE  SEX   AGE   RESTRICT FLANG
          <int> <chr> <chr>       <chr> <chr> <chr> <chr> <chr> <chr> <chr>    <chr>
    1         1 adv7… ADV         u1    S1    NRN   ST    F     4     NONE     EST  
    2         2 adv7… ADV         u1    S2    NS    JU    F     1     NONE     <NA> 
    3         3 adv7… ADV         u1    S1    NRN   ST    F     4     NONE     EST  
    4         4 adv7… ADV         u1    S2    NS    JU    F     1     NONE     <NA> 
    5         5 adv7… ADV         u1    S1    NRN   ST    F     4     NONE     EST  
    6         6 adv7… ADV         u1    S2    NS    JU    F     1     NONE     <NA> 
    # ℹ 1 more variable: text <chr>

``` r
tail(micase_df)
```

    # A tibble: 6 × 12
      unique_id file  speechevent level WHO   NSS   ROLE  SEX   AGE   RESTRICT FLANG
          <int> <chr> <chr>       <chr> <chr> <chr> <chr> <chr> <chr> <chr>    <chr>
    1     53747 tou9… TOU         u2    SU-m  NS    JU    M     1     NONE     <NA> 
    2     53748 tou9… TOU         u2    SU-m  NS    JU    M     1     NONE     <NA> 
    3     53749 tou9… TOU         u2    SU-m  NS    JU    M     1     NONE     <NA> 
    4     53750 tou9… TOU         u2    SU-m  NS    JU    M     1     NONE     <NA> 
    5     53751 tou9… TOU         u2    R1    NS    SG    F     3     NONE     <NA> 
    6     53752 tou9… TOU         u2    SU-m  NS    JU    M     1     NONE     <NA> 
    # ℹ 1 more variable: text <chr>

The file name is included in the data frame so that each utterance can
be connected to the transcription that it came from. The type of speech
event is included so that the data can be analyzed for potential
differences in speech contributions across speech event types. All
speaker information provided by the transcript is retained in the data
frame. Indications of overlapping speech are not included in the data
frame in order to keep the data tidy. Indications of overlaps may be
added at a later time if deemed relevant enough to the analysis to be
included. The decision to separate the utterances into two levels of
utterances is based on the structure of the MICASE transcriptions where
U1 nodes represent a speaker taking a full conversation turn and U2
nodes represent a speaker responding verbally without taking a full
conversation turn.

# Session Info

``` r
sessionInfo()
```

    R version 4.3.2 (2023-10-31)
    Platform: x86_64-apple-darwin20 (64-bit)
    Running under: macOS Monterey 12.6.5

    Matrix products: default
    BLAS:   /Library/Frameworks/R.framework/Versions/4.3-x86_64/Resources/lib/libRblas.0.dylib 
    LAPACK: /Library/Frameworks/R.framework/Versions/4.3-x86_64/Resources/lib/libRlapack.dylib;  LAPACK version 3.11.0

    locale:
    [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

    time zone: America/Detroit
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    other attached packages:
     [1] xml2_1.3.6      lubridate_1.9.3 forcats_1.0.0   stringr_1.5.1  
     [5] dplyr_1.1.4     purrr_1.0.2     readr_2.1.4     tidyr_1.3.0    
     [9] tibble_3.2.1    ggplot2_3.4.4   tidyverse_2.0.0

    loaded via a namespace (and not attached):
     [1] gtable_0.3.4      compiler_4.3.2    tidyselect_1.2.0  scales_1.3.0     
     [5] yaml_2.3.8        fastmap_1.1.1     R6_2.5.1          generics_0.1.3   
     [9] knitr_1.45        munsell_0.5.0     pillar_1.9.0      tzdb_0.4.0       
    [13] rlang_1.1.3       utf8_1.2.4        stringi_1.8.3     xfun_0.42        
    [17] timechange_0.2.0  cli_3.6.2         withr_2.5.2       magrittr_2.0.3   
    [21] digest_0.6.35     grid_4.3.2        rstudioapi_0.15.0 hms_1.1.3        
    [25] lifecycle_1.0.4   vctrs_0.6.5       evaluate_0.23     glue_1.7.0       
    [29] fansi_1.0.6       colorspace_2.1-0  rmarkdown_2.26    tools_4.3.2      
    [33] pkgconfig_2.0.3   htmltools_0.5.7  
