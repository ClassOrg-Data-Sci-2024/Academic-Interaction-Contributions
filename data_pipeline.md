Data-processing Pipeline
================
Jack Rechsteiner
04/22/2024

- [Data overview](#data-overview)
- [Data pipeline](#data-pipeline)
  - [Initial data cleaning](#initial-data-cleaning)
- [Creating a word token data frame](#creating-a-word-token-data-frame)
  - [Counting instances of non-lexicals, backchannels, and
    exclamations](#counting-instances-of-non-lexicals-backchannels-and-exclamations)
- [Turn-taking analysis](#turn-taking-analysis)
  - [Gender](#gender)
  - [Age](#age)
  - [Academic Role](#academic-role)
- [Word count analysis](#word-count-analysis)
  - [Gender](#gender-1)
  - [Age](#age-1)
  - [Academic Role](#academic-role-1)
- [Session Info](#session-info)

``` r
##Set knitr options (show both code and output, show output w/o leading #)
knitr::opts_chunk$set(echo = TRUE, include = TRUE, comment=NA, fig.path = "Images/")

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

#load lme4 and lmerTest
library(lme4)
```

    ## Loading required package: Matrix
    ## 
    ## Attaching package: 'Matrix'
    ## 
    ## The following objects are masked from 'package:tidyr':
    ## 
    ##     expand, pack, unpack

``` r
library(lmerTest)
```

    ## 
    ## Attaching package: 'lmerTest'
    ## 
    ## The following object is masked from 'package:lme4':
    ## 
    ##     lmer
    ## 
    ## The following object is masked from 'package:stats':
    ## 
    ##     step

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
                    speech_event = speechevent_list, 
                    ##'level' to indicate whether it is a first-level or second-level utterance
                    level = "u1", 
                    ##'nodeset' which contains the lists of attributes for speakers
                    nodeset = u1_speaker_nodes, 
                    ##'text' which contains the text from the speaker
                    text = u1_speaker_text) %>% 
  #Using add_row() to add the data from the second-level utterances and set the level to "u2"
  add_row(file = filename_list, 
          speech_event = speechevent_list, 
          level = "u2", 
          nodeset = u2_speaker_nodes, 
          text = u2_speaker_text) %>% 
  #Using unnest() on the columns that contain lists created by map()
  unnest(cols = c(file, speech_event, nodeset, text)) %>% 
  #Using unnest_wider() on the nodeset lists so that the XML attribute titles become columns 
  ##which are filled with the XML attribute values
  unnest_wider(nodeset)

#Renaming columns to be more transparent
micase_df <- micase_df %>% 
  rename(speaker_id = WHO, 
         native_english_status = NSS, 
         academic_role = ROLE, 
         gender = SEX, 
         age_range = AGE, 
         first_language = FLANG) %>% 
  #Replacing value codes with full value names
  mutate(across(speech_event, ~ (str_replace_all(.x, c("ADV" = "Advising Session", 
                                                       "COL" = "Colloquium", 
                                                       "DEF" = "Dissertation Defense", 
                                                       "DIS" = "Discussion Section", 
                                                       "INT" = "Interview", 
                                                       "LAB" = "Lab Section",
                                                       "LEL" = "Large Lecture",
                                                       "LES" = "Small Lecture",
                                                       "MTG" = "Meeting",
                                                       "OFC" = "Office Hours",
                                                       "SEM" = "Seminar",
                                                       "SGR" = "Study Group",
                                                       "STP" = "Student Presentation",
                                                       "SVC" = "Service Encounter",
                                                       "TOU" = "Tour")))),
         across(native_english_status, ~ (str_replace_all(.x, c("^NS$" = "American Speaker",
                                                                "NSO" = "Non-American Speaker",
                                                                "NRN" = "Near Native Speaker",
                                                                "NNS" = "Non-Native Speaker")))),
         #to simplify the categories in the eventual analysis, differences between "junior" and "senior" roles are dropped
         ##for instance, "JU" is code for "junior undergrad" and "SU" is code for "senior undergrad",
         ##but this analysis combines these two categories into "undergrad"
         across(academic_role, ~ (str_replace_all(.x, c("JU" = "Undergrad",
                                                        "SU" = "Undergrad",
                                                        "JG" = "Grad",
                                                        "SG" = "Grad",
                                                        "JF" = "University Employee",
                                                        "SF" = "University Employee",
                                                        "RE" = "Researcher",
                                                        "ST" = "University Employee",
                                                        "VO" = "Visitor/Other",
                                                        "UN" = "Unknown")))),
         across(gender, ~ (str_replace_all(.x, c("F" = "Female",
                                                 "M" = "Male",
                                                 "U" = "Unknown")))),
         across(age_range, ~ (str_replace_all(.x, c("^1$" = "17-23",
                                                    "^2$" = "24-30",
                                                    "^3$" = "31-50",
                                                    "^4$" = "51+",
                                                    "^0$" = "Unknown"))))
  )

#Combining speech event types based on shared characteristics
micase_df <- micase_df %>% 
  mutate(across(speech_event, ~ (str_replace_all(.x, c("Advising Session" = "Institute-led Supplementary", 
                                                       "Colloquium" = "Academic Event", 
                                                       "Dissertation Defense" = "Academic Event", 
                                                       "Discussion Section" = "Student-led Class", 
                                                       "Lab Section" = "Institute-led Class",
                                                       "Large Lecture"= "Institute-led Class",
                                                       "Small Lecture" = "Institute-led Class",
                                                       "Meeting" = "Student-led Supplementary",
                                                       "Office Hours" = "Institute-led Supplementary",
                                                       "Seminar" = "Student-led Class",
                                                       "Study Group" = "Student-led Supplementary",
                                                       "Student Presentation" = "Student-led Class",
                                                       "Service Encounter" = "Academic Event",
                                                       "Tour" = "Academic Event")))))

#Brief samples of the current data frame
head(micase_df)
```

    # A tibble: 6 × 11
      file  speech_event level speaker_id native_english_status academic_role gender
      <chr> <chr>        <chr> <chr>      <chr>                 <chr>         <chr> 
    1 adv7… Institute-l… u1    S1         Near Native Speaker   University E… Female
    2 adv7… Institute-l… u1    S2         American Speaker      Undergrad     Female
    3 adv7… Institute-l… u1    S1         Near Native Speaker   University E… Female
    4 adv7… Institute-l… u1    S2         American Speaker      Undergrad     Female
    5 adv7… Institute-l… u1    S1         Near Native Speaker   University E… Female
    6 adv7… Institute-l… u1    S2         American Speaker      Undergrad     Female
    # ℹ 4 more variables: age_range <chr>, RESTRICT <chr>, first_language <chr>,
    #   text <chr>

``` r
tail(micase_df)
```

    # A tibble: 6 × 11
      file  speech_event level speaker_id native_english_status academic_role gender
      <chr> <chr>        <chr> <chr>      <chr>                 <chr>         <chr> 
    1 tou9… Academic Ev… u2    SU-m       American Speaker      Undergrad     Male  
    2 tou9… Academic Ev… u2    SU-m       American Speaker      Undergrad     Male  
    3 tou9… Academic Ev… u2    SU-m       American Speaker      Undergrad     Male  
    4 tou9… Academic Ev… u2    SU-m       American Speaker      Undergrad     Male  
    5 tou9… Academic Ev… u2    R1         American Speaker      Grad          Female
    6 tou9… Academic Ev… u2    SU-m       American Speaker      Undergrad     Male  
    # ℹ 4 more variables: age_range <chr>, RESTRICT <chr>, first_language <chr>,
    #   text <chr>

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
utterances will also be removed from the dataset. The MICASE manual also
states that utterances with an “SS” value in the `WHO` column are from
two or more speakers in unison, which makes it difficult to link these
contributions to specific speakers. As such, these utterances will be
removed. During the exploration of the data, it was noticed that the
file `mtg999st015` is not easily comparable to the other files in this
dataset as it is a transcript of a forum for international educators
that had exclusively staff participants. All other events in the
“meeting” category are student-led events, so `mtg999st015` will be
omitted from analysis so it does not skew the data for this event type.
`ofc105su068` is also omitted from the analysis as it contains much
redacted speech which strongly skews the contribution counts. Events in
the “interview” category will also be omitted from analysis as they are
also difficult to compare to the rest of the data set due to their
sociolinguistic interview nature.

``` r
#saving changes to a new df so that the original is still around if I need it
micase_df_turns <- micase_df %>% 
  #mutating across the text column to replace whitespace at the beginning and end of lines with nothing,
  #all other whitespace with a single space, all "\n" with nothing, and all "(xx)" with nothing
  mutate(across(text, ~ (str_replace_all(.x, c("^\\s" = "", "\\s$" = "", "\\s+" = " ", "\\n" = "", "\\(xx\\)" = ""))))) %>% 
  #removing all text cells that contain "RESTRICTED" values
  filter(text != "RESTRICTED") %>% 
  #dropping the RESTRICT column because it is not informative with the "RESTRICTED" values removed
  select(!RESTRICT) %>% 
  #removing all rows with speaker_id cells that contain "SS" values
  filter(speaker_id != "SS") %>% 
  #removing data for file `mtg999st015`
  filter(file != "mtg999st015") %>% 
  #removing data for file `ofc105su068`
  filter(file != "ofc105su068") %>% 
  #removing interview events
  filter(speech_event != "Interview")
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
  unnest(cols=c("text")) %>% 
  #remove any rows with empty "text" cells
  filter(text != "")
```

## Counting instances of non-lexicals, backchannels, and exclamations

The MICASE Manual lists all the transcription conventions used for
hesitation and filler words, backchannel cues, exclamations, and
truncated words which allows all these instances to be counted in a
straightforward way. Getting counts of these phenomena will be helpful
in guiding how the analysis handles them.

``` r
#concatenating the regexs to be mapped over to find hesitations, backchannels, exclamations, and truncations
nonlexical_regexs <- c(hesitation = "^(hm|hm'|huh|mm|mhm|uh|um|mkay)[^\\w\\s]?$", 
                       backchannel = "^(okey-doke|okey-dokey|uhuh|yeah|yep|yuhuh|uh'uh|huh'uh|'m'm|huh'uh)[^\\w\\s]?$", 
                       exclamation = "^(ach|ah|ahah|gee|jeez|oh|ooh|oop|oops|tch|ugh|uh'oh|whoa|yay)[^\\w\\s]?$", 
                       truncated = "-[^\\w\\s]?$")

#saving a count of the total cells in micase_df_words
total_word_count <- sum(micase_df_words %>% count())

#creating a word counter function that takes a regex string as input
word_counter <- function(regex_string) {
  micase_df_words %>% 
    #filtering micase_df_words based on cells in the text column that match the regex string
    filter(str_detect(text, regex_string)) %>% 
    #getting a count of the words matched
    count() %>% 
    #creating a column for the percentage of the total word count that are matched strings
    mutate(percentage_of_total = sum(n)/total_word_count)
}

#mapping the concatenated regex strings over the word_counter() function
map_dfr(nonlexical_regexs, word_counter, .id = "category")
```

    # A tibble: 4 × 3
      category        n percentage_of_total
      <chr>       <int>               <dbl>
    1 hesitation  19359             0.0216 
    2 backchannel  8790             0.00980
    3 exclamation  3522             0.00393
    4 truncated   10461             0.0117 

Based on these total counts, exclamations will be removed from the data
frame because they represent less than one percent of the total data.
These word counts are being used as a quantifiable proxy for a speaker’s
level of engagement with a speech event, so truncated words,
hesitations, and filler words will also be removed but backchannel cues
will be retained.

``` r
micase_df_words <- micase_df_words %>% 
  #using filter() to select every row that does not match the three regex strings
  filter(!str_detect(text, nonlexical_regexs["hesitation"]),
         !str_detect(text, nonlexical_regexs["exclamation"]),
         !str_detect(text, nonlexical_regexs["truncated"]))
```

# Turn-taking analysis

Examining contributions in academic interactions by the number of turns
taken will be accomplished with the `micase_df_turns` dataframe that has
been created. The MICASE Manual describes utterances tagged as
second-level as “Backchannel cues from a speaker who doesn’t hold the
floor and unsuccessful attempts to take the floor” (p. 14), so the
initial analysis will only focus on first-level utterances which
represent speakers successfully taking the floor.

``` r
#creating a new df for the analysis of the turn data by percentages
micase_turn_analysis <- micase_df_turns %>% 
  #filtering to just look at first-level utterances
  filter(level == "u1") %>% 
  #adding count column for total turns taken in event
  add_count(file, name = "event_turn_total") %>% 
  #using group_by to organize turns according to the file they come from and the speaker_id
  group_by(file, speaker_id) %>% 
  #adding count column for all turns taken by each speaker in every file
  add_count(name = "speaker_turns") %>% 
  #adding column calculating the the percentage of turns contributed by each speaker in every event
  mutate(percent_turns_contributed = speaker_turns/event_turn_total) %>% 
  #dropping columns that aren't relevant to analysis
  select(!c(level, text)) %>% 
  #using slice to select the first row of each group
  ##essentially providing a single row for each speaker in every interaction
  ##while still retaining the metadata information for the speaker
  slice(1) %>% 
  #filtering out speakers who contributed less than 1% of words to event
  filter(percent_turns_contributed >= 0.01) %>% 
  ungroup()

#creating a data frame where turn contribution is a binary categorical variable for linear modelling
micase_turn_model <- micase_turn_analysis %>% 
  #creating unique speaker ids to use as random effect by uniting file values with speaker_id values
  unite(unique_speaker_id, file, speaker_id) %>% 
  #creating a column for the count of turns a speaker didn't take in the interaction
  mutate(not_speaking = event_turn_total-speaker_turns,
         #mapping the turns not taken to a list column with a number of 0 values equal to the turns not taken
         not_speaking = map(not_speaking, ~ rep(0, .x)),
         #mapping the turns taken to a list column with a number of 1 values equal to the turns taken
         speaker_turns = map(speaker_turns, ~ rep(1, .x)),
         #combining the two list columns into a singular list column
         turn_taken = map2(speaker_turns, not_speaking, c)) %>%
  #dropping all the old count columns and intermediary columns
  select(!c(event_turn_total:not_speaking)) %>% 
  #unnesting the turns_taken column so that speakers have multiple rows where 1 indicates they took a turn
  #and 0 indicates that someone else took a turn
  unnest(cols = c(turn_taken))
```

## Gender

This analysis of the turn-taking data will look at the effects of gender
on turn contributions in interactions.

``` r
#creating a plot for institute-led and student-led events by gender
micase_turn_analysis %>% 
  #filtering out speakers with unknown gender and "Academic Event" speech events
  filter(gender != "Unknown", speech_event != "Academic Event") %>% 
  #creating plot with 'gender' on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = gender, y = percent_turns_contributed)) + 
  #specifying plot type as geom_boxplot()
  geom_boxplot() +
  #changing axis labels to be more informative and adding title
  labs(x= "Gender", y = "Percent turns taken in interaction", title = "Institute- and Student-led Events") +
  #making title look nicer
  theme(plot.title = element_text(face = "bold", size = 12, hjust=0.5)) +
  #setting y-axis limits to go from 0 to 1 to make graphs comparable
  ylim(0, 1)
```

![](Images/turn-taking%20gender%20analysis-1.png)<!-- -->

``` r
#creating a plot for academic events by gender
micase_turn_analysis %>% 
  #filtering out speakers with unknown gender and selecting "Academic Event" speech event
  filter(gender != "Unknown", speech_event == "Academic Event") %>% 
  #creating plot with 'gender' on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = gender, y = percent_turns_contributed)) + 
  #specifying plot type as geom_boxplot()
  geom_boxplot() +
  #changing axis labels to be more informative and adding title
  labs(x= "Gender", y = "Percent turns taken in interaction", title = "Academic Events") +
  #making title look nicer
  theme(plot.title = element_text(face = "bold", size = 12, hjust=0.5)) +
  #setting y-axis limits to go from 0 to 1 to make graphs comparable
  ylim(0, 1)
```

![](Images/turn-taking%20gender%20analysis-2.png)<!-- -->

``` r
#linear model analysis of gender in the two event categories
micase_turn_model %>% 
  #filtering out speakers of unknown gender
  filter(gender != "Unknown") %>% 
  #replacing all speech_event strings that start with "Institute" or "Student" with "Class/Supplementary"
  mutate(
    across(speech_event, ~ str_replace_all(.x, c("^Institute.+" = "Class/Supplementary", 
                                                 "^Student.+" = "Class/Supplementary"))
    ),
    #encoding gender as a factor so it can be releveled with the reference level as "Male"
    gender = (factor(gender) %>% relevel(gender, ref = "Male")),
    #encoding speech_event as a factor so it can be releveled with the reference level as "Class/Supplementary"
    speech_event = (factor(speech_event) %>% relevel(speech_event, ref = "Class/Supplementary"))
  ) %>% 
  #linear model to see the effects of gender * speech_event on turn_taken with a random effect for speaker_id
  lmer(turn_taken ~ gender * speech_event + (1|unique_speaker_id), data = .) %>% 
  summary()
```

    Linear mixed model fit by REML. t-tests use Satterthwaite's method [
    lmerModLmerTest]
    Formula: turn_taken ~ gender * speech_event + (1 | unique_speaker_id)
       Data: .

    REML criterion at convergence: 131218.9

    Scaled residuals: 
        Min      1Q  Median      3Q     Max 
    -2.5661 -0.4204 -0.1377 -0.0585  3.2931 

    Random effects:
     Groups            Name        Variance Std.Dev.
     unique_speaker_id (Intercept) 0.01773  0.1332  
     Residual                      0.09029  0.3005  
    Number of obs: 296884, groups:  unique_speaker_id, 591

    Fixed effects:
                                              Estimate Std. Error         df
    (Intercept)                               0.108482   0.008622 585.284521
    genderFemale                              0.011401   0.011625 585.488594
    speech_eventAcademic Event                0.019476   0.024936 589.585330
    genderFemale:speech_eventAcademic Event  -0.018005   0.039772 586.670010
                                            t value Pr(>|t|)    
    (Intercept)                              12.582   <2e-16 ***
    genderFemale                              0.981    0.327    
    speech_eventAcademic Event                0.781    0.435    
    genderFemale:speech_eventAcademic Event  -0.453    0.651    
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Correlation of Fixed Effects:
                (Intr) gndrFm spc_AE
    genderFemal -0.742              
    spch_vntAcE -0.346  0.256       
    gndrFml:_AE  0.217 -0.292 -0.627

## Age

This analysis of the turn-taking data will look at the effects of age on
turn contributions in interactions.

``` r
#creating plots for contributions in class/Supplementary events and academic events by age range
micase_turn_analysis %>% 
  #filtering out speakers with unknown ages
  filter(age_range != "Unknown") %>% 
  #replacing all speech_event strings that start with "Institute" or "Student" with "Class/Supplementary"
  mutate(
    across(speech_event, ~ str_replace_all(.x, c("^Institute.+" = "Class/Supplementary", 
                                                 "^Student.+" = "Class/Supplementary")))
  ) %>% 
  #creating plot with age_range on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = age_range, y = percent_turns_contributed)) + 
  #specifying plot type as geom_boxplot()
  geom_boxplot() +
  #using facet_wrap() to get separate plots for class/Supplementary events and academic events
  facet_wrap(vars(speech_event)) +
  #changing axis labels to be more informative
  labs(x= "Age Range", y = "Percent turns taken in interaction") 
```

![](Images/turn-taking%20age%20analysis-1.png)<!-- -->

## Academic Role

This analysis of the turn-taking data will look at the effects of
academic role on turn contributions in interactions.

``` r
#creating a plot for institute-led and student-led events by academic role
micase_turn_analysis %>% 
  #filtering out speakers with visitor roles and unknown roles, and presentations
  filter(academic_role != "Visitor/Other", academic_role != "Unknown", speech_event != "Academic Event") %>% 
  #creating plot with 'academic_role' on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = academic_role, y = percent_turns_contributed)) + 
  #specifying plot as geom_jitter() with a width of .3
  geom_jitter(width = .3) +
  #creating facets for each speech event and setting the direction to vertical
  facet_wrap(vars(speech_event), dir="v") +
  #changing axis labels to be more informative
  labs(x= "Academic Role", y = "Percent turns taken in interaction")
```

![](Images/turn-taking%20academic%20role%20analysis-1.png)<!-- -->

``` r
#concatenating the event types to be used to calculate average contributions in each event type
eventtypes <- c(Institute_led_Class = "Institute-led Class", 
                Student_led_Class = "Student-led Class", 
                Institute_led_Supplementary = "Institute-led Supplementary", 
                Student_led_Supplementary = "Student-led Supplementary", 
                Academic_Event = "Academic Event")

#mapping the concatenated event types over functions to calculate the event averages for the micase_turn_analysis dataframe
map_dfr(eventtypes, ~ micase_turn_analysis %>% 
          #selecting undergrad speakers
          filter(academic_role == "Undergrad") %>% 
          #filtering to look at one specific event type
          filter(speech_event == .x) %>% 
          #pulling the numbers out of the column so it plays nice with mean()
          pull(percent_turns_contributed) %>% 
          #calculating the mean
          mean() %>% 
          #turning the result into a tibble so it plays nice with map_dfr()
          as_tibble(), 
        #setting the name of the output dataframe's id column to "Event Type"
        .id = "Event Type")
```

    # A tibble: 5 × 2
      `Event Type`                 value
      <chr>                        <dbl>
    1 Institute_led_Class         0.0671
    2 Student_led_Class           0.0473
    3 Institute_led_Supplementary 0.0857
    4 Student_led_Supplementary   0.178 
    5 Academic_Event              0.130 

# Word count analysis

Examining contributions in academic interactions by the number of words
spoken will be accomplished with the `micase_df_words` dataframe that
has been created.

``` r
#creating a new df for the analysis of the turn data
micase_word_analysis <- micase_df_words %>% 
  add_count(file, name = "event_word_total") %>% 
  #using group_by to organize turns according to the file they come from and the speaker_id
  group_by(file, speaker_id) %>% 
  #adding count column for all turns taken by each speaker in every file
  add_count(name = "speaker_words") %>% 
  #adding column calculating the the percentage of turns contributed by each speaker in every event
  mutate(percent_words_contributed = speaker_words/event_word_total) %>% 
  #dropping columns that aren't relevant to analysis
  select(!text) %>% 
  #using slice to select the first row of each group
  ##essentially providing a single row for each speaker in every interaction
  ##while still retaining the metadata information for the speaker
  slice(1) %>% 
  #filtering out speakers who contributed less than 1% of words to event
  filter(percent_words_contributed >= 0.01) %>% 
  ungroup()

#creating a data frame where word contribution is a binary categorical variable for linear modelling
micase_word_model <- micase_word_analysis %>% 
  #creating unique model ids to use as random effect by uniting file values with speaker_id values
  unite(unique_speaker_id, file, speaker_id) %>% 
  #creating a column for the count of words a speaker didn't say in the interaction
  mutate(not_speaking = event_word_total-speaker_words,
         #mapping the words not spoken to a list column with a number of 0 values equal to the words not spoken
         not_speaking = map(not_speaking, ~ rep(0, .x)),
         #mapping the words spoken to a list column with a number of 1 values equal to the words spoken
         speaker_words = map(speaker_words, ~ rep(1, .x)),
         #combining the two list columns into a singular list column
         word_spoken = map2(speaker_words, not_speaking, c)) %>%
  #dropping all the old count columns and intermediary columns
  select(!c(event_word_total:not_speaking)) %>% 
  #unnesting the word_spoken column so that speakers have multiple rows where 1 indicates they spoke a word
  #and 0 indicates that someone else spoke a word
  unnest(cols = c(word_spoken))
```

## Gender

This analysis of the word count data will look at the effects of gender
on word contributions in interactions.

``` r
#creating a plot for institute-led and student-led events by gender
micase_word_analysis %>% 
  #filtering out speakers with unknown gender and "Academic Event" speech events
  filter(gender != "Unknown", speech_event != "Academic Event") %>% 
  #creating plot with 'gender' on the x-axis and percent_words_contributed on the y-axis
  ggplot(aes(x = gender, y = percent_words_contributed)) + 
  #specifying plot type as geom_boxplot()
  geom_boxplot() +
  #changing axis labels to be more informative and adding title
  labs(x= "Gender", y = "Percent words spoken in interaction", title = "Institute- and Student-led Events") +
  #making title look nicer
  theme(plot.title = element_text(face = "bold", size = 12, hjust=0.5)) +
  #setting y-axis limits to go from 0 to 1 to make graphs comparable
  ylim(0, 1)
```

![](Images/word%20count%20gender%20analysis-1.png)<!-- -->

``` r
#creating a plot for academic events by gender
micase_word_analysis %>% 
  #filtering out speakers with unknown gender and selecting "Academic Event" speech event
  filter(gender != "Unknown", speech_event == "Academic Event") %>% 
  #creating plot with 'gender' on the x-axis and percent_words_contributed on the y-axis
  ggplot(aes(x = gender, y = percent_words_contributed)) + 
  #specifying plot type as geom_boxplot()
  geom_boxplot() +
  #changing axis labels to be more informative and adding title
  labs(x= "Gender", y = "Percent words spoken in interaction", title = "Academic Events") +
  #making title look nicer
  theme(plot.title = element_text(face = "bold", size = 12, hjust=0.5)) +
  #setting y-axis limits to go from 0 to 1 to make graphs comparable
  ylim(0, 1)
```

![](Images/word%20count%20gender%20analysis-2.png)<!-- -->

``` r
#linear model analysis of gender in the two event categories
micase_word_model %>% 
  #filtering out speakers of unknown gender
  filter(gender != "Unknown") %>% 
  #replacing all speech_event strings that start with "Institute" or "Student" with "Class/Supplementary"
  mutate(
    across(speech_event, ~ str_replace_all(.x, c("^Institute.+" = "Class/Supplementary", 
                                                 "^Student.+" = "Class/Supplementary"))
    ),
    #encoding gender as a factor so it can be releveled with the reference level as "Male"
    gender = (factor(gender) %>% relevel(gender, ref = "Male")),
    #encoding speech_event as a factor so it can be releveled with the reference level as "Class/Supplementary"
    speech_event = (factor(speech_event) %>% relevel(speech_event, ref = "Class/Supplementary"))
  ) %>% 
  #linear model to see the effects of gender * speech_event on word_spoken with a random effect for unique_speaker_id
  ##also specifying the optimizer as bobyqa because nloptwrap was having convergence issues
  lmer(word_spoken ~ gender * speech_event + (1|unique_speaker_id), data = ., control = lmerControl(optimizer = "bobyqa")) %>% 
  summary()
```

    Linear mixed model fit by REML. t-tests use Satterthwaite's method [
    lmerModLmerTest]
    Formula: word_spoken ~ gender * speech_event + (1 | unique_speaker_id)
       Data: .
    Control: lmerControl(optimizer = "bobyqa")

    REML criterion at convergence: 1784499

    Scaled residuals: 
        Min      1Q  Median      3Q     Max 
    -3.3144 -0.2966 -0.1210 -0.0529  3.5431 

    Random effects:
     Groups            Name        Variance Std.Dev.
     unique_speaker_id (Intercept) 0.04275  0.2068  
     Residual                      0.07805  0.2794  
    Number of obs: 6193187, groups:  unique_speaker_id, 487

    Fixed effects:
                                              Estimate Std. Error         df
    (Intercept)                               0.142303   0.014733 483.348340
    genderFemale                             -0.007578   0.019683 483.358984
    speech_eventAcademic Event                0.035067   0.044707 483.347110
    genderFemale:speech_eventAcademic Event  -0.005480   0.070852 483.363181
                                            t value Pr(>|t|)    
    (Intercept)                               9.659   <2e-16 ***
    genderFemale                             -0.385    0.700    
    speech_eventAcademic Event                0.784    0.433    
    genderFemale:speech_eventAcademic Event  -0.077    0.938    
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Correlation of Fixed Effects:
                (Intr) gndrFm spc_AE
    genderFemal -0.749              
    spch_vntAcE -0.330  0.247       
    gndrFml:_AE  0.208 -0.278 -0.631

## Age

This analysis of the word count data will look at the effects of age on
word contributions in interactions.

``` r
#creating plots for contributions in class/Supplementary events and academic events by age range
micase_word_analysis %>% 
  #filtering out speakers with unknown ages
  filter(age_range != "Unknown") %>% 
  #replacing all speech_event strings that start with "Institute" or "Student" with "Class/Supplementary"
  mutate(
    across(speech_event, ~ str_replace_all(.x, c("^Institute.+" = "Class/Supplementary", 
                                                 "^Student.+" = "Class/Supplementary")))
  ) %>% 
  #creating plot with age_range on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = age_range, y = percent_words_contributed)) + 
  #specifying plot type as geom_boxplot()
  geom_boxplot() +
  #using facet_wrap() to get separate plots for class/Supplementary events and academic events
  facet_wrap(vars(speech_event)) +
  #changing axis labels to be more informative
  labs(x= "Age Range", y = "Percent words spoken in interaction") 
```

![](Images/word%20count%20age%20analysis-1.png)<!-- -->

## Academic Role

This analysis of the word count data will look at the effects of
academic role on word contributions in interactions.

``` r
#creating a plot for institute-led and student-led events
micase_word_analysis %>% 
  #filtering out speakers with visitor roles and unknown roles, and presentations
  filter(academic_role != "Visitor/Other", academic_role != "Unknown", speech_event != "Academic Event") %>% 
  #creating plot with 'academic_role' on the x-axis and percent_turns_contributed on the y-axis
  ggplot(aes(x = academic_role, y = percent_words_contributed,  group = academic_role)) + 
  #specifying plot as geom_jitter() with a width of .3
  geom_jitter(width = .3) +
  #creating facets for each speech event and setting the direction to vertical
  facet_wrap(vars(speech_event), dir="v") +
  #changing axis labels to be nicer
  labs(x= "Academic Role", y = "Percent words spoken in interaction")
```

![](Images/word%20count%20academic%20role%20analysis-1.png)<!-- -->

``` r
#mapping the concatenated event types over functions to calculate the event averages for the micase_word_analysis dataframe
map_dfr(eventtypes, ~ micase_word_analysis %>% 
          #selecting undergrad speakers
          filter(academic_role == "Undergrad") %>% 
          #filtering to look at one specific event type
          filter(speech_event == .x) %>% 
          #pulling the numbers out of the column so it plays nice with mean()
          pull(percent_words_contributed) %>% 
          #calculating the mean
          mean() %>% 
          #turning the result into a tibble so it plays nice with map_dfr()
          as_tibble(), 
        #setting the name of the output dataframe's id column to "Event Type"
        .id = "Event Type")
```

    # A tibble: 5 × 2
      `Event Type`                 value
      <chr>                        <dbl>
    1 Institute_led_Class         0.0531
    2 Student_led_Class           0.0401
    3 Institute_led_Supplementary 0.0595
    4 Student_led_Supplementary   0.179 
    5 Academic_Event              0.244 

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
     [1] lmerTest_3.1-3  lme4_1.1-35.3   Matrix_1.6-4    xml2_1.3.6     
     [5] lubridate_1.9.3 forcats_1.0.0   stringr_1.5.1   dplyr_1.1.4    
     [9] purrr_1.0.2     readr_2.1.4     tidyr_1.3.0     tibble_3.2.1   
    [13] ggplot2_3.4.4   tidyverse_2.0.0

    loaded via a namespace (and not attached):
     [1] utf8_1.2.4          generics_0.1.3      stringi_1.8.3      
     [4] lattice_0.22-6      hms_1.1.3           digest_0.6.35      
     [7] magrittr_2.0.3      evaluate_0.23       grid_4.3.2         
    [10] timechange_0.2.0    fastmap_1.1.1       fansi_1.0.6        
    [13] scales_1.3.0        numDeriv_2016.8-1.1 cli_3.6.2          
    [16] rlang_1.1.3         munsell_0.5.0       splines_4.3.2      
    [19] withr_2.5.2         yaml_2.3.8          tools_4.3.2        
    [22] tzdb_0.4.0          nloptr_2.0.3        minqa_1.2.6        
    [25] colorspace_2.1-0    boot_1.3-28.1       vctrs_0.6.5        
    [28] R6_2.5.1            lifecycle_1.0.4     MASS_7.3-60.0.1    
    [31] pkgconfig_2.0.3     pillar_1.9.0        gtable_0.3.4       
    [34] glue_1.7.0          Rcpp_1.0.12         highr_0.10         
    [37] xfun_0.42           tidyselect_1.2.0    rstudioapi_0.15.0  
    [40] knitr_1.45          farver_2.1.1        htmltools_0.5.7    
    [43] nlme_3.1-164        labeling_0.4.3      rmarkdown_2.26     
    [46] compiler_4.3.2     
