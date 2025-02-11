---
title: Converting an example motion tracking dataset for sharing in BIDS
tags: [example, bids, sharing, motion, qualisys, optotrak, xsens]
---

# Converting an example motion tracking dataset for sharing in BIDS

{% include markup/danger %}
The [BIDS standard](https://bids.neuroimaging.io) does currently not specify how to represent motion tracking data. This example - and the support that is implemented in the **[data2bids](https://github.com/fieldtrip/fieldtrip/blob/release/data2bids.m)** function - should be considered as a preliminary proposal to help researchers with their existing data.

See [this issue](https://github.com/bids-standard/bids-specification/issues/443) on GitHub and [this Google doc](https://docs.google.com/document/d/1iaaLKgWjK5pcISD1MVxHKexB3PZWfE2aAC5HF_pCZWo/edit?usp=sharing) with the draft specification that is being discussed.
{% include markup/end %}

There are numerous companies that manufacture research-oriented motion capture systems, such as Polhemus, Qualisys, NDI Polaris, X-Sens, etc. Furthermore, there are multiple technologies that are used for motion tracking, such as optical cameras (often with IR markers), electromagnetic tracking, or using inertial measurement units (IMUs). Optical and electromagnetic tracking systems result in measurements that can directly be interpreted as the position of the marker, which changes over time. Systems based on IMUs record signals from multiple accelerometers, gyroscopes, and (sometimes) magnetometers; the raw data from these systems requires further processing before it can be interpreted as position.

Motion tracking data - optionally in relation to the presentation of stimulus material and responses (e.g., button presses) that are given by the subject - can be stored in the BIDS representation in line with the specification of [behavioral data](https://bids-specification.readthedocs.io/en/stable/04-modality-specific-files/07-behavioral-experiments.html). We store the continuously motion tracking data in tab-separated-value format (TSV) in the `_motion.tsv` file. The events corresponding to stimuli and responses are stored in the `_events.tsv` file, similar as for other types of data in BIDS.

{% include markup/info %}
Besides storing the output of the motion capture system, the position of the (optical) markers on the body, the frame of reference (coordinate system) and the units should be documented in the sidecar JSON file. This can be added to the `cfg.motion` field to **[data2bids](https://github.com/fieldtrip/fieldtrip/blob/release/data2bids.m)** function, e.g., as `cfg.motion.MarkerPositions` and `cfg.motion.MotionCoordinateSystem`.
{% include markup/end %}

## Qualisys

[Qualisys](https://www.qualisys.com) consists of a camera-based motion capture system. _It would be nice to have some more details here on the system and corresponding software, please contribute your knowledge via [GitHub](https://github.com/fieldtrip/website)._

### Example

The example that we present here was recorded using a [Qualisys](https://www.qualisys.com) camera-based motion capture system. The data was exported from the proprietary Qualisys `.qtm` format to the standard biomechanics `.c3d` format (see [this link](https://www.c3d.org) for the standard) and to the `.tsv` (tab-separated-values) format; data in both exported formats can directly be read and processed by FieldTrip.

{% include markup/success %}
The original data for the following example and the converted BIDS representation are available from our [FTP server](ftp://ftp.fieldtriptoolbox.org/pub/fieldtrip/example/bids_motion/).
{% include markup/end %}

```
cfg = [];

cfg.InstitutionName             = 'Radboud University';
cfg.InstitutionalDepartmentName = 'Donders Institute for Brain, Cognition and Behaviour';
cfg.InstitutionAddress          = 'Kapittelweg 29, 6525 EN, Nijmegen, The Netherlands';

% required for dataset_description.json
cfg.dataset_description.Name                = 'Motion capture example';
cfg.dataset_description.BIDSVersion         = 'unofficial extension';

% optional for dataset_description.json
cfg.dataset_description.License             = 'n/a';
cfg.dataset_description.Authors             = 'n/a';
cfg.dataset_description.Acknowledgements    = 'n/a';
cfg.dataset_description.Funding             = 'n/a';
cfg.dataset_description.ReferencesAndLinks  = 'n/a';
cfg.dataset_description.DatasetDOI          = 'n/a';

cfg.dataset = './original/self_test_30April2015_ADA.c3d';  % exported from Qualisys
% cfg.dataset = './original/self_test_30April2015_ADA.tsv'; % alternative export format

cfg.bidsroot = './bids';  % write to the present working directory
cfg.datatype = 'motion';
cfg.sub = 'S01';

% these are general fields
cfg.Manufacturer           = 'Qualisys';
cfg.ManufacturersModelName = '6+';

cfg.TaskDescription = 'The subject was making some hand movements for a short test recording';
cfg.task = 'handmovement';

data2bids(cfg);
```

## Optotrak

The Optotrak software, NDI First Principles, automatically exports a collection file, an experiment file, a tool definition file, and two raw data files. One of the raw data files contains the Optotrak data, which consists of the 3D data from the attached markers, and the other raw data file contains the ODAU data. The ODAU is a separate unit of the Optotrak system that digitizes additional analog inputs, e.g., from analog inputs to synchronize the data to an external system. Additionally, the software can convert the data and export it in the following formats: NDI 3D, NDI Odau, C3D, AII to ASCII, or NDI 6D. The export format can be specified in the software prior to starting the recording, or the data can be converted offline.

Experimental events, such as the beginning of a trial or the onset of a stimulus, are usually captured by sending a signal to one of the analog channels of the ODAU unit of the Optotrak system. This can be done by sending for example a step function or a pulse with a given amplitude and duration. This analog signal differs from experimental events in for example the data from Presentation software, as it is saved as a continuous analog stream of data (similar to the 3D marker data) instead of saving the onset and offset of a certain event in a separate events file.

### Example

The following example demonstrates how to convert an Optotrak dataset to BIDS. The data was donated for testing purposes; the recording details are not really known.

The data was exported using the Optotrak software to TSV (tab separated values) files. Inspecting them in a text editor shows that they are comma separated, so a file extension of CSV (comma separated values) would have made more sense. There are two exported files, one for the motion data and one for the analog channels.

FieldTrip does not have direct support for the Optotrak file format, nor for the specific format in which the data is exported to the TSV files. However, it is trivial to read the TSV files using MATLABs **[readtable](https://nl.mathworks.com/help/matlab/ref/readtable.html.m)** function. Subsequently, we will use the approach explained [here](/faq/how_can_i_import_my_own_dataformat/#circumvent-the-fieldtrip-reading-functions) to convert the tabular data to a FieldTrip data structure, similar to what would have been returned by **[ft_preprocessing](https://github.com/fieldtrip/fieldtrip/blob/release/ft_preprocessing.m)**. The data2bids function can take this as input and will export it in the right format, including the required metadata.

{% include markup/success %}
The original data for the following example and the converted BIDS representation are available from our [FTP server](ftp://ftp.fieldtriptoolbox.org/pub/fieldtrip/example/bids_motion/).
{% include markup/end %}

```
% although the files have the extension tsv, they use a comma as separator
% they also contain 4 heading lines

table_3d   = readtable('./original/dataOptotrak/HandChoiceSwitch_EEG_01_001_3d.tsv', 'FileType', 'text', 'Delimiter', ',', 'HeaderLines', 4);
table_0dau = readtable('./original/dataOptotrak/HandChoiceSwitch_EEG_01_001_Odau_1.tsv', 'FileType', 'text', 'Delimiter', ',', 'HeaderLines', 4);

% the last column is empty
table_3d = table_3d(:,1:end-1);
table_0dau = table_0dau(:,1:end-1);

% plot(table_3d.Frame, table_3d.Marker_1X)
% plot(table_0dau.Frame, table_0dau.Analog_1)

data_3d = [];
data_3d.label    = table_3d.Properties.VariableNames;
data_3d.trial{1} = table2array(table_3d)';
data_3d.time{1}  = (table_3d.Frame')/250;

data_0dau = [];
data_0dau.label    = table_0dau.Properties.VariableNames;
data_0dau.trial{1} = table2array(table_0dau)';
data_0dau.time{1}  = (table_0dau.Frame')/250;

%%

% both files have the Frame as the first column, hence as the first channel
% ensure they are equal
assert(isequal(data_3d.trial{1}(1,:), data_0dau.trial{1}(1,:)))

% remove the Frame channel
cfg = [];
cfg.channel = {'all', '-Frame'};
data_0dau = ft_selectdata(cfg, data_0dau);

% combine the two datasets
cfg = [];
data_combined = ft_appenddata(cfg, data_3d, data_0dau);

%% visualize the data to see what is happening

cfg = [];
cfg.viewmode = 'vertical';
cfg.ylim = 'maxabs';
cfg.preproc.demean = 'yes';
cfg.channel = {'all', '-Frame'};
ft_databrowser(cfg, data_combined);

%% export the data to BIDS

cfg = [];

cfg.InstitutionName             = 'Radboud University';
cfg.InstitutionalDepartmentName = 'Donders Institute for Brain, Cognition and Behaviour';
cfg.InstitutionAddress          = 'Kapittelweg 29, 6525 EN, Nijmegen, The Netherlands';

% required for dataset_description.json
cfg.dataset_description.Name                = 'Motion capture example';
cfg.dataset_description.BIDSVersion         = 'unofficial extension';

% optional for dataset_description.json
cfg.dataset_description.License             = 'n/a';
cfg.dataset_description.Authors             = 'n/a';
cfg.dataset_description.Acknowledgements    = 'n/a';
cfg.dataset_description.Funding             = 'n/a';
cfg.dataset_description.ReferencesAndLinks  = 'n/a';
cfg.dataset_description.DatasetDOI          = 'n/a';

cfg.dataset = './original/dataOptotrak/HandChoiceSwitch_EEG_01_001_3d.tsv';  % exported from Qualisys
% cfg.dataset = './original/self_test_30April2015_ADA.tsv'; % alternative export format

cfg.bidsroot = './bids';  % write to the present working directory
cfg.datatype = 'motion';
cfg.sub = 'S01';

% these are general fields
cfg.Manufacturer           = 'Optotrak';
cfg.ManufacturersModelName = 'Unknown';

cfg.TaskDescription = 'The subject was switching between hand movements';
cfg.task = 'HandChoiceSwitch';

data2bids(cfg, data_combined);
```

## XSens

[XSens](http://www.xsens.com/) makes IMU-based motion capture systems that are used in the animation industry and in research. Their MVN Analyze system comprises full-body sensor systems and aquisition and analysis software. By default the software stores the data in the proprietary MVN file format, but it allows the data to be exported to C3D and MVNX formats, which are supported by FieldTrip. See also the [getting started](/getting_started/xsens/) documentation on this system.

### Example

Here is a short example that demonstrates how XSens data can be converted to BIDS, starting from either C3D files or from MVNX files.

{% include markup/success %}
The original data for the following example and the converted BIDS representation are available from our [FTP server](ftp://ftp.fieldtriptoolbox.org/pub/fieldtrip/example/bids_motion/).
{% include markup/end %}

```

c3dfile  = './original/example1.c3d';
mvnxfile = './original/example2.mvnx';

%%

cfg = [];

cfg.InstitutionName             = 'Radboud University';
cfg.InstitutionalDepartmentName = 'Donders Institute for Brain, Cognition and Behaviour';
cfg.InstitutionAddress          = 'Kapittelweg 29, 6525 EN, Nijmegen, The Netherlands';

% required for dataset_description.json
cfg.dataset_description.Name                = 'Motion capture example';
cfg.dataset_description.BIDSVersion         = 'unofficial extension';

% optional for dataset_description.json
cfg.dataset_description.License             = 'n/a';
cfg.dataset_description.Authors             = 'n/a';
cfg.dataset_description.Acknowledgements    = 'n/a';
cfg.dataset_description.Funding             = 'n/a';
cfg.dataset_description.ReferencesAndLinks  = 'n/a';
cfg.dataset_description.DatasetDOI          = 'n/a';

% these are general fields
cfg.Manufacturer           = 'XSens';
cfg.ManufacturersModelName = 'Awinda';

cfg.TaskDescription = 'Walking in a corridor';
cfg.task = 'walking';

cfg.bidsroot = './bids';  % write to the present working directory
cfg.datatype = 'motion';

%% we can export c3d files, here for subject one
cfg.sub = 'S01';
cfg.participants.age = 32;
cfg.participants.gender = 'm';
cfg.participants.height = 186;
cfg.dataset = c3dfile;

data2bids(cfg);

%% we can also export mvnx files, here for subject two
cfg.sub = 'S02';
cfg.participants.age = 28;
cfg.participants.gender = 'f';
cfg.participants.height = 167;
cfg.dataset = mvnxfile;

data2bids(cfg);
```
