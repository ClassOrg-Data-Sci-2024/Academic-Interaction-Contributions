# Progress Report

## Entry #1: 02/20/2024

I have created the GitHub repository for my final project on [individual contributions in academic interactions](https://github.com/ClassOrg-Data-Sci-2024/Academic-Interaction-Contributions) and have created or modified the following files:

-   [`README.md`](README.md) now includes the title of the project, my name, and a short summary of the project.

-   [`.gitignore`](.gitignore) now includes the macOS .gitignore template and to ignore files in the local `private/` directory.

-   [`LICENSE.md`](LICENSE.md) has been created, but it only includes placeholder text.

-   [`project_plan.md`](project_plan.md) has been created and contains information on the project's scope, as well as plans for data manipulation and analysis.

-   `progress_report.md` has been created with the information that you are now reading (to paraphrase Virginia Woolf — yes, dear reader, this document is alive, but don't worry about it).

## 1st Progress Report: 03/15/2024

I have acquired the 75 XML files that will be used in my project, which are located in the [Data folder](Data/) of this repository.
An overview of this data, including information on where this data comes from as well as the size and makeup of the data set, can be found in the [Data Overview section](data_pipeline.md#data-overview) of the [`data_pipeline.md` file](data_pipeline.md). 
The `data_pipeline.md` file additionally contains the code for [my current data pipeline](data_pipeline.md#data-pipeline).

In addition to acquiring the data and adding files to this project's repository, the steps I have accomplished so far are:

- Learning how to use the [`xml2` package](https://cran.r-project.org/web/packages/xml2/index.html) to read and extract data from XML files.

- Learning XPath syntax in order to specify the data to be extracted from the XML files.

- Constructing code for a data import process using `xml2` and the [`purrr` package](https://purrr.tidyverse.org/) to efficiently process the 75 files in the data set.

- Creating a tibble from the XML data and unnesting the nodes to represent the data in a tidy format.

### Next Steps

The next steps that I believe would be most useful are cleaning up the text data and transforming some of the metadata to be more informative.
In regard to cleaning up the data, there are currently `\n` strings in some first-level utterances that will need to be removed. 
These strings seem to be artifacts caused by removing the second-level utterance child nodes, so it would be worthwhile to investigate if any other anomalies appear in the text due to the data import process.
It would also be helpful to find any documentation on the transcription conventions for [MICASE](https://quod.lib.umich.edu/cgi/c/corpus/corpus), if possible.
There are certain notations that occur regularly, such as underscores attached to words and speech transcribed as "(xx)", that I would like to clarify the meaning of in order to determine how they should be handled in the analysis.
Thankfully, transforming the metadata will be more straightforward, as the full forms of the attribute codes can be found on the [MICASE search page](https://quod.lib.umich.edu/cgi/c/corpus/corpus?c=micase;page=simple).
It should be possible to make the metadata more transparent and informative by converting the abbreviated codes from the XML files to the corresponding descriptive attributes present on the MICASE website.

## 2nd Progress Report: 03/31/2024

As was mentioned in the previous progress report, one goal was to find documentation of the MICASE transcription conventions.
The [MICASE Manual](https://ca.talkbank.org/access/0docs/MICASE.pdf) is still thankfully available from [TalkBank](https://talkbank.org/), and the information that it provides has been useful for exploring and processing the transcription data from the corpus.
The work accomplished since the previous progress report has primarily focused on cleaning and reorganizing the data, beginning initial analyses of the data, and finalizing the license for this project.
This has included:

- Removing strings created in the process of moving data from XML format to tibble format.

- Removing words that were transcribed as "completely unintelligible".

- Removing data that has been redacted from the corpus.

- Creating one dataframe for analyzing turn-taking and another dataframe for analyzing word counts.

- Getting a count of the number of hesitation and filler words, backchannel cues, exclamations, and truncated words in the data.

- Deriving data for the number and calculated percentage of turns taken by each participant in each interaction.

- Creating a preliminary `geom_count()` plot for the relation between participant gender and the percentage of turns participants took.

- Finalizing the licensing terms as seen in [`LICENSE.md`](LICENSE.md).

- Adding a subtitle to the [`data_pipeline.md` file](data_pipeline.md) for this project which specifies that it is the existing script file from the first progress report and is being updated with new code.

- Adding the MICASE copyright statement to this project's [`README.md`](README.md).

### Sharing Scheme

I have decided to share the transcript data used for this project in its entirety.
The MICASE copyright statement states that "The database is freely available at the MICASE website for study, teaching, and research purposes, and copies of the transcripts may be distributed, as long as either this statement of availability or the citation given below appears in the text".
The full MICASE copyright statement can be read in the [MICASE Manual](https://ca.talkbank.org/access/0docs/MICASE.pdf) and on the [`README.md`](README.md) for this project.
The rationale behind including the transcripts is that the XML files used are freely available online from MICASE, potentially sensitive information has already been redacted from the transcripts, and the size of the files allows them to be easily shared.
As such, sharing the data promotes Open Data while presenting minimal potential for causing harm or creating risk.

### Licensing Decisions

I chose to license this project under the GNU General Public License v3.0, as it seemed to be the most fitting license to open source sharing.
It was important to me that the components of this project are able to be used, modified, and shared by other interested parties.
However, I also wanted to prevent against the possibility of closed source versions being distributed, so that any future usage of the content presented here would remain open to others.
The GNU General Public License v3.0 accomplishes these goals which makes it a reasonable license for this project.