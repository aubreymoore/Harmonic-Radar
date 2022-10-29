---
title: 'Harmonic Radar: Drone Trial 2022-06-09'
toc: structure
author: Aubrey Moore and Glenn Dulla
date: Last revised June 14, 2022

documentclass: scrartcl
geometry: margin=2cm
lang: en

pagesize: letter
fontsize: 12pt
graphics: yes
colorlinks: true
---

<!--
pandoc techreport.md -f markdown -o techreport.pdf --number-sections
mupdf techreport.pdf
-->

**GitHub Repostory:** <https://github.com/aubreymoore/2022-06-09-drone>

\newpage

# Objective

The objective of this trial was was to map the location of a single harmonic radar target placed beneath a linear transect flown by a drone.  

# Methods

A RECCO hand-held harmonic radar transceiver was suspended with rope about 1.5 feet beneath the landing struts of a DJI AGRIS MG-1 drone. A target consisting of 2 harmonic radar tags with antennae placed at right angles (\autoref{target}) was positioned in an open field and the drone was programmed to fly along a line which crossed this position.

\begin{figure}[]
\centering
\includegraphics[width=0.5\textwidth]{target.png}
\caption{Target.}
\label{target}
\end{figure}


## Harmonic radar recording

The RECCO hand-held harmonic radar device generates an audio signal to indicate that a reflection from a harmonic radar tag has been detected. The amplitude of this signal is maximum when the receiving antennae points at the target and it increases as the target gets closer. A human operator locates the direction of a tag by directional scanning with the antenna while monitoring the signal using a built-in speaker or headphones. In this application, we point the receiving antenna straight down and record the signal by connecting a small digital audio field recorder (ZOOM F2) to the audio jack. The F2 records monophonic 32-bit floating point WAV files at a rate of 48,000 samples per second.

An Audacity screenshot displays the waveform and spectrogram of the record created during the trial (\autoref{audacity}). The WAV file was processed using a Jupyter notebook which performed the following steps.

* Noise reduction: Background noise was removed from the signal by the [noisereduce Python library](https://pypi.org/project/noisereduce/). \autoref{audacity-noise-reduction} is an Adacity screenshot showing the waveform and spectrogram after noise reduction.
* Data reduction: The mean absolute amplitude (MAA) of the signal was calculated for each second within the WAV file. MAA is used as a measure of signal strength.

\begin{figure}[p]
\centering
\includegraphics[]{audacity.png}
\caption{Audacity.}
\label{audacity}
\end{figure}

\begin{figure}[p]
\centering
\includegraphics[]{audacity-noise-reduction.png}
\caption{Audacity-noise-reduction.}
\label{audacity-noise-reduction}
\end{figure}

\clearpage

## GPS track recording

The flight data log was downloaded as FLY275.DAT from the drone. This file was parsed using a free Java program named DatCon using parameters recorded in \autoref{DataCon}. DataCon creates three files:

* a CSV containing 1,275 columns. I set the sample rate to one row per second (1 Hz).
* a text file containing event messages.
* a KML file containing latitude and longitude of points along the drone's track. \autoref{GoogleEarth} is a screenshot of the total flight displayed using Google Earth.

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{DataCon.png}
\caption{DataCon.}
\label{DataCon}
\end{figure}

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{GoogleEarth.png}
\caption{Google Earth.}
\label{DataCon}
\end{figure}



## Merging the two recordings

The following steps were used to map harmonic radar signal strength.

**Step 1** The audio record was broken into one-second segments. Each segment was represented by a row containing the following variables.

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{df-audio.png}
\caption{Audio dataframe.}
\label{df-audio}
\end{figure}

**Step 2** The flight data record, FLY275.DAT, was parsed into one-second segments with each represented by a row containing the following fields.

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{df-flight.png}
\caption{Flight dataframe.}
\label{df-flight}
\end{figure}

\clearpage

**Step 3**

The Python pandas module makes it extremely easy to merge the two dataframes. The merge method recognizes that the *timestamp* column is present in both datasets. Data from rows are combined wherever *timestamp* values are equal.

df_merged = df_audio.merge(df_flight)

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{df-merged.png}
\caption{Merged dataframe.}
\label{df-merged}
\end{figure}

\newpage

# Results and Discussion

## Flight path

\autoref{flightpath} shows the drone's flight path. The flight was comprised of three segments.

1. **Outbound flight.** The drone took off from the westernmost extent of the plot. It then flew rapidly to the easternmost extent of the plot.
2. **Programmed mission.** The drone flew at slow speed from the eastern extent of the plot to the southernmost extent of the plot.
3. **Return to home and landing.** Upon completion of the mission, the drone ascended to about 35 meters above the ground and flew rapidly to home where it landed.

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{flightpath.png}
\caption{Flight path.}
\label{flightpath}
\end{figure}

\clearpage
## Harmonic radar

\autoref{latlonmaa} shows the strength of harmonic radar reflections recorded during each second while the drone was flying from east to west along the programmed linear transect. Each point represents signal strength integrated over one second. The bright yellow point indicating the highest signal strength probably indicates the location of the single harmonic radar target placed within the transect. Unfortunately, the location of the was not recorded. Aubrey took an image of the target with his smartphone assuming that the image would be georeferenced. But the GPS device on his phone had been turned off.

\begin{figure}[]
\centering
\includegraphics[width=\textwidth]{latlonmaa.png}
\caption{latlonmaah}
\label{latlonmaa}
\end{figure}

## Notes

* The original plan was to merge the audio record with the drone's GPS data by matching timestamps. A better approach may be to place one or more harmonic radar targets at known locations within the drone's search pattern.

* Cepstrum analysis may be useful for increasing the signal to noise ratio of the audio signal. See <https://www.kuniga.me/blog/2021/12/11/pitch-via-cepstrum.html>
