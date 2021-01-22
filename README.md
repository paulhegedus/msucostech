
# MSUCOSTECH

Montana State University Environmental Laboratory Shiny application for
creating automated data reports from raw Costech ECS 4010 data.

Users must have R downloaded on the computer they are running the
application from **(?)**.

The application can be found at **URL**

### Inputs

The application takes a .xlsx with Result Table, Chromatogram Data,
Chromatogram Graph, and Summary Table tabs. When the user browses to
their raw Costech data the Summary Table of the Excel file is imported.
The user also must provide inputs for the following parameters in the
model; + Detection Limit Factor: To estimate ‘below detection limit’,
what is the factor to divide the low calibration standard by? Default is
2.5. + Total Nitrogen Uncertainty Factor: Estimate for the uncertainty
of total nitrogen %. Provide as decimal, default is 0.1. + Total Carbon
Uncertainty Factor: Estimate for the uncertainty of total Carbon %.
Provide as decimal, default is 0.05. + Nitrogen Consensus Value:
Consensus value for nitrogen derived from prior Costech standards,
divided from the measured standards to evaluate new measurements with
prior Costech data. Default is 0.15. + Carbon Consensus Value: Consensus
value for carbon derived from prior Costech standards, divided from the
measured standards to evaluate new measurements with prior Costech data.
Default is 2.

The output Excel spreadsheet has a sheet called User Inputs that
contains columns with the name of each input above, the abbreviation
used in the code, and the entry the user provided. If the user does not
provide one of the inputs, the User Inputs table will display the
default value used for the calculations. See the workflow for where each
input is used in the code.

### Outputs

When raw data and inputs are provided, the user must select the ‘Create
Report’ button. The output data report will be **automatically
downloaded upon completion.** The output Excel sheet has tabs for
**Results, Summary Table, Calibration Range, Standards, and User
Inputs**. The Results table contains columns for the sample ID, the
percent total nitrogen and carbon, uncertainties for nitrogen and
carbon, and a flag for each indicating whether the observation was above
or below the quantifiable limit (‘aql’, ‘bql’), below the detection
limit (‘bdl’), or within detectable limits (‘ok’). The uncertainties are
calculated by multiplying the weight percent by the user specified
uncertainty factor. The Summary Table contains the unmodified raw data.
The Calibration Range table contains the low and high quantifiable
limits for nitrogen and carbon, calculated from the minimum and maximum
nitrogen and carbon values measured in the acetanalide samples. This
table also contains the detection limit for nitrogen and carbon,
calculated by dividing the low quantifiable limit by the user specified
detection limit factor. The Standards table contains the sample ID,
weight (mg), weight percent, and retention time for nitrogen and carbon
for each observation. This table also has columns for the mean, standard
deviation, and coefficient of variation for the measured standards and a
consensus metric for each observation, calculated as the measured value
divided by the user specified consensus value for nitrogen and carbon.

### Workflow

The following image shows the workflow for creating the automated report
from the Costech. The workflow shows the inputs used in the code and
Shiny application, the functions used in the application code, and the
tables that compose the exported report.

<div class="figure" style="text-align: center">

<img src="/Users/PaulBriggs/Box/Hegedus/Projects/EAL/MSUCOSTECH/MSUCOSTECH_Reports/www/msucostech_workflow.png" alt="Workflow for the MSU EAL Costech automated data report. Green shapes in the image below characterize inputs, with ovals representing inputs that are also used in the Shiny application. Functions used in the application are shown in blue and temporary data is represented by yellow shapes. Sheets in the exported Excel report are shown in coral. All of the inputs are included in the User Inputs table of the output." width="75%" />

<p class="caption">

Workflow for the MSU EAL Costech automated data report. Green shapes in
the image below characterize inputs, with ovals representing inputs that
are also used in the Shiny application. Functions used in the
application are shown in blue and temporary data is represented by
yellow shapes. Sheets in the exported Excel report are shown in coral.
All of the inputs are included in the User Inputs table of the output.

</p>

</div>

The workflow begins by importing raw data. This uses either the
specified folder path and filename or just the filename, assuming it
includes the full path, to extract the ‘Summary Table’ sheet of the
specified Excel spreadsheet. Alternatively, this file is browsed to in
the Shiny application, and data is imported the same way. When the user
elects to create the data report, the raw data is immediately set aside
for export. The Standards table is created using the raw data and
consensus values for nitrogen and carbon. Next, the Calibration Range
table is generated using a subset of the raw data including only the
Acetanilide samples and the user specified detection limit factor. After
the Calibration Range table is generated, the Results table is created
using the calibration values and the user specified uncertainty factors.
Finally, the data is packaged into an Excel file and exported.

The scripts for the application and the code for creating the report are
contained in the MSUCOSTECH\_Reports folder of this repository. The
**MSUCOSTECH\_Reports\_App.R** contains the application deployment and
**MSUCOSTECH\_fxns.R** contains the supporting code. The following
sections describe the arguments, process, and output for each function;
+ importRawDat(): Requires a path to the Excel file exported from the
Costech ECS 4010. This function imports the Summary Table sheet, sets
the column names, and returns the raw data frame. This requires the
Costech outputs to remain standard. + genStandardsTab(): This function
subsets Standard D samples from the raw data. Currently this process is
performed via pattern matching expected codes used for Standard D
samples. The SampleID column of the raw data is searched for ‘Std\_D’,
‘Standard’, and ‘StdD’. This is not case sensitive and requires a mild
standardization of sample IDs. The mean and standard deviation are
calculated for nitrogen and carbon and added as columns to the subset.
The coefficient of variation for both elements are then calculated as
the standard deviation divided by the mean. A consensus metric is then
derived by dividing the weight percent by the user specified consensus
value for each element. The columns of the subset are grouped by element
and then returned to the user. + genCalRangeTab(): This function subsets
Acetanilide samples from the raw data. Currently this process is
performed via pattern matching expected codes used for Standard D
samples. The SampleID column of the raw data is searched for
‘Acetanilide’, ‘Acet’, ‘Acetan’, ‘Acetanalide’, ‘analide’. This is not
case sensitive and requires a mild standardization of sample IDs. An
empty Calibration Range table is generated and filled in with the
following calculations. The low and high calibration ranges for each
element are derived by the minimum and maximum weights in the
Acetanilide subset. The DetectionLimit is calculated by dividing the low
calibration range by the user specified detection limit factor. This may
be a temporary process until a data-driven approach is developed. After
the Calibration Range table has been completed, it is returned to the
user. + genResultsTab(): This function uses the identifiers for Standard
D and Acetanilide samples to take all rows that aren’t these or a
bypass, identified by ‘Bypass’ or ‘By pass’ in the SampleID. The
SampleID, weight and weight percents are the columns taken from the raw
data during the subsetting process. The nitrogen and carbon percent
uncertainty are calculated and added to the table by multiplying the
weight percent by the nitrogen or carbon uncertainty factor that was
specified by the user. For each element, the flag is derived by querying
if the weight falls first below the detection limit, then within the
bounds of the quantifiable limits. Observations are labeled ‘bdl’ if the
measurement falls below the detection limit, ‘bql’ if the measurement
falls below the lower quantifiable limit, ‘aql’ if the measurment
exceeds the upper quantifiable limit, or ‘ok’ if it falls within the
quantifiable limits. The weight columns are removed from the Results
table, the columns are organized by element, and the Results table is
returned to the user. + exportReport(): This function takes the User
Inputs, Summary Table, Standards, Calibration Range, and Results table
and puts them into an Excel spreasheet that is exported to the user’s
file system in the Downloads folder.
