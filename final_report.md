# Individual Contributions to Academic Interactions

**Jack Rechsteiner**

April 23, 2022

## Introduction

Historically, discourse analysis has often focused on small sets of data to make qualitative claims. 
Integrating data science methods with discourse analysis approaches makes it more feasible to do quantitative analyses on corpora that would be too difficult or time-consuming to do without computer assistance. 
Performing these quantitative analyses is one avenue for providing further evidence and support for the claims made by qualitative work (Johnstone, 2018).
However, it is important for these quantitative analyses to be informed by previous qualitative work so that the methodological and analytic decisions are motivated by attested observations.

Previous discourse analysis work has observed that the contributions of an individual in an interaction are affected by social factors. 
This has led to a definition of *asymmetrical talk* as talk where participants have unequal positions in conversations, due to differences in power, status, and/or control (Cameron, 2001).
These differences frequently arise from the social positions of conversational participants, as social position often determines one's access to power, status, and control.
Talk that occurs in an institutional context, such as in academic settings, often shows more asymmetrical distributions of conversational contributions than "ordinary talk" does (Cameron, 2001, p. 100).
The asymmetry in institutional contexts an be understood as resulting from differences in the institutional position of participants, in addition to differences in social position.
Analyzing asymmetry in talk can provide insight into the distribution of power and inequality in these contexts.
This project adds to this body of work with a quantitative analysis of the amount of speech contributed by individuals in academic interactions and the effects that institutional position and social position have on individual contribution levels.

## Research Questions and Hypotheses

Using data from the Michigan Corpus of Academic Spoken English (MICASE), this research project aimed to investigate three research questions:

1. How does the institutional position of an individual affect their contributions in an academic interaction?

1. How does the social position of an individual affect their contributions in an academic interaction?

1. How do institutional position and social position interact with each other in academic interactions?

I hypothesized that:

1. Speech event type will be related to differences in contribution levels based on the institutional expectations of the speech event. For instance, class events are more institutionally structured so they will show a greater inequality in contribution distributions than supplementary events ("supplementary" is used in this project to refer to academic non-class events, such as advising sessions and office hours).

1. Social factors will be more relevant to contribution levels when institutional positions are less relevant to an event.

## Data: Michigan Corpus of Academic Spoken English (MICASE)

MICASE is a corpus that was created by the English Language Institute at the University of Michigan between 1997 and 2002.
The corpus consists of transcriptions and XML files for nearly 200 hours of academic speech from 152 different speech events.
MICASE defines academic speech as “speech which occurs in academic settings” which makes this data a suitable fit for investigating asymmetrical talk in institutional contexts (2002, p. 4).
The XML files from the corpus contain metadata on the speech event and the participants involved, as well as the entire transcript. 
This metadata includes information on the type of speech event, the level of interactivity, the number of participants, the English language proficiency of speakers, the academic roles of participants, and the gender and age group for each speaker. 

Specifically, this project uses data from the 75 events that were tagged as “Highly Interactive” or “Mostly Interactive” which represent roughly 6,360 minutes of recorded speech interactions.
However, it was decided that five of these events were not sufficiently comparable to the other events and were excluded from the analysis.
Three of the omitted events were sociolinguistic research interviews and thus represented a different conversational mode than the other events.
The fourth omitted event was a forum for international educators that had exclusively staff participants from the "meeting" category of speech events; all the other events in the "meeting" category are student-led events, so this staff forum was omitted in order to not skew the data for the "meeting" event category.
The fifth omitted event was an "office hours" category event that had much of the speech redacted, and this redaction caused the file to not be suitable for analysis.
This resulted in a total of 70 different academic speech events being used in the analysis.

As one of the research questions of this project is investigating the effects of social position on conversational contributions, it is important to note that MICASE attempted “to get approximately equal amounts of speech from male and female speakers within each academic division” (2002, p. 5).
Therefore, it is reasonable to expect that the social effects of gender on contribution amounts will be diminished in the data, as the creators of the corpus have already controlled for contribution by gender.
This data is still viable for investigating more nuanced questions of gender effects, such as if female students have higher levels of contributions in classes led by female faculty compared to classes led by male faculty.
Unfortunately, questions of this nature are beyond the scope of this project, and the analysis of gender presented later in this report should be understood in light of these circumstances.

## Methods

As a way to bridge previous qualitative observations about asymmetrical talk with a quantitative approach, this project uses two measures to quantify "conversational contribution": the number of turns taken by an individual and the number of words spoken by an individual.
This operationalization of contribution reflects the claim from Clark & Schaefer (1989) that contributions are constructed of two phases. 
The first is the presentation phase where a contributor presents an utterance and the second is the acceptance phase where the recipient(s) provide evidence that they understand the utterance.
The presentation of an utterance necessarily requires taking a turn, but the acceptance phase does not always take the form of a turn; evidence of understanding can be shown with non-verbal continued attention signals, like eye contact and head nodding, or with linguistic acts are not conversational turns, like backchannel cues.
In this way, a measurement of the number of turns taken serves as an approximation of the amount of times a speaker presents an utterance in the first phase and the amount of times they take a conversational turn to provide evidence in the second phase.
and the measurement of words spoken gives a more general sense of an individual's verbal engagement with a conversation.
While both these measures are unable to account for aspects like the relevance or quality of the utterances, they allow for some general insights into who is allowed to talk how much in which contexts.

To obtain these measurements, the MICASE XML transcript files were imported into R and turned into a tibble data frame.
This initial data frame contained a row for each utterance in the transcript with information about the file name, the type of speech event, whether the utterance was a conversational turn or not, the speaker's ID code, and demographic information for the speaker. 
Demographic information included the speaker's English speaking status, academic role, gender, and first language other than English (when applicable).
Academic role designations were collapsed from the original MICASE designations into the groups of undergraduate student, graduate student, university employee (faculty and staff), researcher, and visitor/other.
The event types from MICASE were combined into broader categories based on shared characteristics, resulting in the following distribution of event types:

- 18 Institute-led Class Events (large lectures, small lectures, and lab sections)

- 16 Institute-led Supplementary Events (advising sessions and office hours)

- 16 Student-led Class Events (discussion sections, seminars, and student presentations)

- 13 Student-led Supplementary Events (student meetings and study groups)

- 7 Academic Events (colloquia/workshops, dissertation defenses, service encounters, and tours)

The initial data frame was utilized to create a turn analysis data frame containing all utterances coded as conversational turns and a word token data frame that split utterances so that each word was its own row.
After assessing the presence of hesitation and filler words, backchannel cues, exclamations, and truncated words in the word token data frame, exclamations were removed as they represented less than one percent of the total data. 
Because the word token data frame was created as a quantifiable proxy for a speaker’s level of engagement with a speech event, truncated words, hesitations, and filler words were also removed but backchannel cues were retained.

The steps of the data cleaning and manipulation for this project can be seen in full in the (`data_pipeline.md`)[data_pipeline.md] file.

## Analysis

The analysis presented below looks at how turn and word contributions are separately affected by one institutional factor, academic role, and two social factors, gender and age.

*Contributions by academic role across events types*

![](Images/turn-taking%20academic%20role%20analysis-1.png)
**Figure 1**

Figure 1, as seen above, shows the distribution of the percentage of turns taken in an interaction by academic role across the Institute-Led and Student-Led Class events and Supplementary events.
The Institute-Led Class events, Institute-Led Supplementary events, and Student-Led Class events show a pattern for undergraduate students to generally be in the 0% to 20% range, with some variation. 
Graduate students follow a similar trend, although the Institute-Led Supplementary events shows a cluster of graduate students in the 0% to 20% range and another cluster in the 40% to 60% range.
University employees in the Institute-Led events overall have percentage rates between 30% and 60%.
However, university employees in the Student-Led Class events have percentages that are more dispersed throughout the 0% to 60% range even if students are remaining in the 0% to 20% range.
The Student-Led Supplementary events show the most equal distribution between academic roles, with all three groups being dispersed throughout the 0% to 40% range.

![](/Images/word count academic role analysis-1.png)
**Figure 2**

Figure 2 plots the distribution of the percentage of words spoken in an interaction, but is otherwise the same as Figure 1 in showing the percentage by academic role across the Institute-Led and Student-Led Class events and Supplementary events.
The general patterns in Figure 2 are similar to those in Figure 1, with some differences.
The word count percentages for graduate students and university employees show greater variance across events than the turn taking percentages, while the word count percentages for undergraduate students remain in the 0% to 20% range for all events other than Student-Led Supplementary events.

To investigate this data further, averages were calculated for undergraduate turn and word contribution percentages across the event categories of Institute-Led Class, Student-Led Class, Institute-Led Supplementary, Student-Led Supplementary, and Academic Event.
These averages are shown below in Table 1.

| Event type                  | Undergraduate turn average | Undergraduate word average  |
|-----------------------------|----------------------------|-----------------------------|
| Institute-led Class         | 6.7%                       | 5.3%                        |
| Student-led Class           | 4.7%                       | 4%                          |
| Institute-led Supplementary | 9.5%                       | 7.3%                        |
| Student-led Supplementary   | 17.8%                      | 17.9%                       |
| Academic Event              | 13%                        | 24.4%                       |
**Table 1**

The first four sets of averages show that the percentage distributions for undergraduate students seen in Figure 1 and Figure 2 are similar for the two contribution measurements.
However, there is an unexpected disparity between the measurement percentages for Academic Event.

*Gender differences in contributions*

![](/Images/turn-taking gender analysis-1.png)
**Figure 3**

MICASE mentions that the corpus attempted to get the same amount of speech from male and female speakers.
This is borne out in the analysis, as seen in Figure 3, as there is little difference in the percentage of turns taken by female and male speakers in Institute-Led and Student-Led events.

![](/Images/turn-taking gender analysis-2.png)
**Figure 4**

Figure 4 appears to show male speakers having slightly higher percentages of turn taking than female speakers in Academic Events.

However, this difference was not found to be statistically significant, as seen in Table 2.

| Fixed effect                             | Estimate | Std. error  | P value      |
|------------------------------------------|----------|-------------|--------------|
| (Intercept)                              | 0.11     | 0.01        | <0.001***    |
| genderFemale                             | 0.01     | 0.01        | 0.33         |
| speech_eventAcademic Event               | 0.02     | 0.02        | 0.44         |
| genderFemale:speech_eventAcademic Event  | -0.02    | 0.04        | 0.65         |
**Table 2**

A linear mixed effects model was run with amount of turns taken as the dependent variable, fixed effects for the interaction of gender and speech event type, and a random effect for participant.
The results showed that only the intercept had a statistically significant P value.
The formula used for the model was `lmer(turn_taken ~ gender * speech_event + (1|unique_speaker_id))`.

![](/Images/word count gender analysis-1.png)
**Figure 5**

The analysis of gender effects on word contributions similarly supports the case that MICASE recorded equal amounts of speech from male and female speakers.
Figure 5 shows very similar percentages of words spoken in an interaction for female and male speakers in Institute-Led and Student-Led events.

![](/Images/word count gender analysis-2.png)
**Figure 6**

The slight difference in turn taking percentages seen in Figure 4 is not present for the word taken analysis, as shown in Figure 6.

Additionally, the statistics for the word token analysis confirm this lack of gender effect, as seen in Table 3.

| Fixed effect                             | Estimate | Std. error  | P value      |
|------------------------------------------|----------|-------------|--------------|
| (Intercept)                              | 0.14     | 0.01        | <0.001***    |
| genderFemale                             | -0.01    | 0.02        | 0.70         |
| speech_eventAcademic Event               | 0.04     | 0.04        | 0.43         |
| genderFemale:speech_eventAcademic Event  | -0.01    | 0.07        | 0.94         |
**Table 3**

A linear mixed effects model was run with number of words spoken as the dependent variable, fixed effects for the interaction of gender and speech event type, and a random effect for participant.
Again, only the intercept had a statistically significant P value.
The formula used for the model was `lmer(word_spoken ~ gender * speech_event + (1|unique_speaker_id))`.

*Age differences in contributions*

![](/Images/turn-taking age analysis-1.png)
**Figure 7**

Figure 7 presents the distribution of percentage of turns taken in an interaction by age range across the categories of Academic Events and Classes + Supplementary events.

![](/Images/word count age analysis-1.png)
**Figure 8**

Comparatively, Figure 8 presents the distribution of percentage of words spoken in an interaction by age range across the categories of Academic Events and Classes + Supplementary events.

Both figures show a general tendency for contribution percentages to be relatively similar across age ranges in Academic Events, regardless of contribution metric.
Both figures also show contribution percentages increasing as age range increases for both turns taken and words spoken.

## Conclusion

This project investigates how conversational contributions by an individual in an academic interaction are conditioned by institutional positions and social positions.
Two hypotheses were proposed at the start of this project.
The first was that the category of speech events would be related to differences in contribution levels based on the institutional expectations of the speech event.
This hypothesis is supported by the analysis of the data which shows that Institute-Led events have more inequality in turn and word contributions than Student-Led events, with Student-Led Supplementary events having the most equal distribution of contributions across academic roles.
The second hypothesis was that social factors will be more relevant to contribution levels when institutional positions are less relevant to an event.
This was tested by comparing Class and Supplementary events to the Academic Event category.
Gender was not seen to affect contribution rates between event types, but this is to be expected with the controlled gender balance in MICASE that was previously noted. 
This second hypothesis seems to be directly refuted by the analysis of the effect that age range has on contribution rates.
For Class and Supplementary events, contribution rates for turns and words increased along with age but this effect was not seen for Academic Event.
Even with assuming there is a correlation between age and institutional position, the second hypothesis would expect that Academic Event some stratification due to the differences in social position between younger and older speakers.
Outside of these hypotheses, this project has also demonstrated two potential methods for quantifying a "contribution" in an interaction and how decisions about quantification can affect the results of an analysis.
The work done in this project is only a small step into the space of quantitative conversation analysis and there are still many topics that can be explored using the Michigan Corpus of Academic Spoken English.
Potentially fruitful avenues for future research could include an investigation of the effects that a speaker's English speaking status has on their conversational contributions, as well as examining the intersections between institutional positions and social positions that exist in this data.

### Project history and process

As I look back on the history and process of this project, I'm happy with how things turned out even if there were some nights where I felt ready to throw in the towel.
The learning curve for converting XML files into tibble data frames was a bit tricky. 
I was ultimately able to figure out the functions needed to have the data format I wanted though.
The amount of metadata tagging in MICASE greatly facilitated analyzing the data for this project, but there were hiccups with some of the files which led to them being omitted as detailed in the [methods](final_report.md#Methods) section of the report.
The granularity of the MICASE tags also led to an iterative process of reassessing the categories used in the analysis to make sure they made sense while also consolidating the data being compared.
Within the analysis portion, there were issues with attempting to do a Poisson regression with the data as a count variable but this was resolved by transforming the data frame so that the count values were expanded into binary categorical variables.
Thanks to this project, and the experience of overcoming these difficulties, I have grown a lot as a data scientist and I look forward to continuing the eternal work-in-progress that is research.

## References

Cameron, D. (2001). *Working with spoken discourse*. SAGE.
Clark, H., & Schaefer, E. (1989). Contributing to discourse. *Cognitive Science, 13*(2), 259–294. 
Johnstone, B. (2018). *Discourse analysis* (Third edition). John Wiley & Sons, Inc.
Simpson, R., Lee, D., & Leicher, S. (2002). *MICASE Manual*. Ann Arbor, Michigan, USA; English Language Institute, The University of Michigan. 