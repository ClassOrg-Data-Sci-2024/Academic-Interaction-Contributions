Data-processing Pipeline
================
Jack Rechsteiner
03/15/2024

- [Data overview](#data-overview)
- [Data pipeline](#data-pipeline)
  - [Initial data cleaning](#initial-data-cleaning)
- [Creating a word token data frame](#creating-a-word-token-data-frame)
  - [Counting instances of non-lexicals, backchannels, and
    exclamations](#counting-instances-of-non-lexicals-backchannels-and-exclamations)
- [Turn-taking analysis](#turn-taking-analysis)
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

## Initial data cleaning

Separating the two utterance levels caused “/n” strings to be created in
first-level utterances at positions where second-level utterances were
originally nested. These strings should be removed as they are not part
of the data being analyzed. Additionally, the [MICASE
Manual](https://ca.talkbank.org/access/0docs/MICASE.pdf) explains in
Table 3-2 that words that were “completely unintelligible” were
transcribed as two x’s in parentheses. These will also be removed from
the data, as it is difficult to quantify the conversational contribution
of a word or words that are not known. It was also discovered that
participants who have values “ALL” or “CITE” values in `RESTRICT` have
had their speech redacted from the corpus. Due to this, these restricted
utterances will also be removed from the dataset.

``` r
#saving changes to a new df so that the original is still around if I need it
micase_df_turns <- micase_df %>% 
  #mutating across the text column to replace whitespace at the beginning and end of lines with nothing,
  #all other whitespace with a single space, all "\n" with nothing, and all "(xx)" with nothing
  mutate(across(text, ~ (str_replace_all(.x, c("^\\s" = "", "\\s$" = "", "\\s+" = " ", "\\n" = "", "\\(xx\\)" = ""))))) %>% 
  #removing all text cells that contain "RESTRICTED" values
  filter(text != "RESTRICTED") %>% 
  #dropping the RESTRICT column because it is not informative with the "RESTRICTED" values removed
  select(!RESTRICT)
```

# Creating a word token data frame

The current data frame is structured in a way that is useful to analyze
turn-taking among conversational participants. In addition to this, a
longer data frame will be created where each word token has its own cell
in the data frame in order to analyze the number of words spoken by each
conversation participant.

``` r
micase_df_words <- micase_df_turns %>% 
  #split the strings in the text column by whitespace
  mutate(across(text, ~ (strsplit(.x, "\\s")))) %>% 
  #unnest the column so that every word is its own cell
  unnest(cols=c("text"))
```

## Counting instances of non-lexicals, backchannels, and exclamations

The MICASE Manual lists all the transcription conventions used for
hesitation and filler words, backchannel cues, exclamations, and
truncated words which allows all these instances to be counted in a
straightforward way. Getting counts of these phenomena will be helpful
in guiding how the analysis handles them.

``` r
#saving a regex string that captures all the MICASE hesitation conventions
hesitation_regex <- "^(hm|hm’|huh|mm|mhm|uh|um|mkay)[^\\w\\s]?$"

#saving a regex string that captures all the MICASE backchannel cue conventions
backchannel_regex <- "^(okey-doke|okey-dokey|uhuh|yeah|yep|yuhuh|uh’uh|huh’uh|‘m’m|huh’uh)[^\\w\\s]?$"

#saving a regex string that captures all the MICASE exclamation conventions
exclamation_regex <- "^(ach|ah|ahah|gee|jeez|oh|ooh|oop|oops|tch|ugh|uh’oh|whoa|yay)[^\\w\\s]?$"

#saving a regex string that captures the MICASE truncated word convention
truncated_regex <- "-[^\\w\\s]?$"

#concatenating the regexs to be mapped over
nonlexical_regexs <- c(hesitation_regex, backchannel_regex, exclamation_regex, truncated_regex)

#saving a count of the total cells in micase_df_words
total_word_count <- sum(micase_df_words %>% count())

#creating a word counter function that takes a regex string as input
word_counter <- function(regex_string) {
  micase_df_words %>% 
    #filtering micase_df_words based on cells in the text column that match the regex string
    filter(str_detect(text, regex_string)) %>% 
    #getting a count of the words matched
    count() %>% 
    #creating a column that includes the regex string that was searched
    #and a column for the percentage of the total word count that are matched strings
    mutate(regex_searched = regex_string, percentage_of_total = sum(n)/total_word_count)
}

#mapping the concanated regex strings over the word_counter() function
map(nonlexical_regexs, ~ word_counter(.x))
```

    [[1]]
    # A tibble: 1 × 3
          n regex_searched                               percentage_of_total
      <int> <chr>                                                      <dbl>
    1 20896 "^(hm|hm’|huh|mm|mhm|uh|um|mkay)[^\\w\\s]?$"              0.0224

    [[2]]
    # A tibble: 1 × 3
          n regex_searched                                       percentage_of_total
      <int> <chr>                                                              <dbl>
    1  9426 "^(okey-doke|okey-dokey|uhuh|yeah|yep|yuhuh|uh’uh|h…              0.0101

    [[3]]
    # A tibble: 1 × 3
          n regex_searched                                       percentage_of_total
      <int> <chr>                                                              <dbl>
    1  3689 "^(ach|ah|ahah|gee|jeez|oh|ooh|oop|oops|tch|ugh|uh’…             0.00395

    [[4]]
    # A tibble: 1 × 3
          n regex_searched percentage_of_total
      <int> <chr>                        <dbl>
    1 10859 "-[^\\w\\s]?$"              0.0116

# Turn-taking analysis

Examining contributions in academic interactions by the number of turns
taken will be accomplished with the `micase_df_turns` dataframe that has
been created. The MICASE Manual describes utterances tagged as
second-level as “Backchannel cues from a speaker who doesn’t hold the
floor and unsuccessful attempts to take the floor” (p. 14), so the
initial analysis will only focus on first-level utterances which
represent speakers successfully taking the floor.

``` r
micase_df_turns %>% 
  #filtering to just look at first-level utterances
  filter(level == "u1") %>% 
  #adding count column for total turns taken in event
  add_count(file, name = "event_turn_total") %>% 
  #using group_by to organize turns according to the file they come from and the ID value from WHO column
  group_by(file, WHO) %>% 
  #adding count column for all turns taken by each speaker in every file
  add_count(name = "speaker_turns") %>% 
  #adding column calculating the the percentage of turns contributed by each speaker in every event
  mutate(percent_turns_contributed = speaker_turns/event_turn_total) %>% 
  #dropping columns that aren't relevant to analysis
  select(-c(unique_id, level, text)) %>% 
  #using slice to select the first row of each group
  ##essentially providing a single row for each speaker in every interaction
  ##while still retaining the metadata information for the speaker
  slice(1) %>% 
  #creating geom_count() plot with SEX on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = SEX, y = percent_turns_contributed)) + 
  geom_count()
```

![](data_pipeline_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

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
     [1] gtable_0.3.4      highr_0.10        compiler_4.3.2    tidyselect_1.2.0 
     [5] scales_1.3.0      yaml_2.3.8        fastmap_1.1.1     R6_2.5.1         
     [9] labeling_0.4.3    generics_0.1.3    knitr_1.45        munsell_0.5.0    
    [13] pillar_1.9.0      tzdb_0.4.0        rlang_1.1.3       utf8_1.2.4       
    [17] stringi_1.8.3     xfun_0.42         timechange_0.2.0  cli_3.6.2        
    [21] withr_2.5.2       magrittr_2.0.3    digest_0.6.35     grid_4.3.2       
    [25] rstudioapi_0.15.0 hms_1.1.3         lifecycle_1.0.4   vctrs_0.6.5      
    [29] evaluate_0.23     glue_1.7.0        farver_2.1.1      fansi_1.0.6      
    [33] colorspace_2.1-0  rmarkdown_2.26    tools_4.3.2       pkgconfig_2.0.3  
    [37] htmltools_0.5.7  
