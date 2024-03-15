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
