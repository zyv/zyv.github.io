\documentclass[12pt,a4paper]{article}

\usepackage{cmap}
\usepackage[utf8]{inputenc}
\usepackage[english]{babel}
\usepackage{lmodern}
\usepackage[pdftex]{hyperref}

\hypersetup{
    colorlinks,
    citecolor=blue,
    filecolor=blue,
    linkcolor=blue,
    urlcolor=blue
}

\usepackage{caption}

\title{Installing Oracle Solaris 10 on Sun Blade 1000}
\author{Yury V. Zaytsev}
\date{\today}

\begin{document}

\section*{Installing Oracle Solaris 10 on Sun Blade 1000}

\section{Introduction}

So, you are a happy owner that wants to get latest and greatest Oracle Solaris running on a legendary Sun Blade 1000 workstation with glorious double UltraSPARC III CPUs and a fair 1 Gb of RAM? Follow on!

\section{Hardware requirements}

\begin{itemize}

	\item Check the version of OpenBoot firmware that your blade is equipped with. There's a known issue with OpenBoot 4.2 which prevents the workstations, having Toshiba DVD-ROM installed from booting from DVD media.

This is a highly annoying problem. In general, two kinds of advices can be found on the net.

	\begin{itemize}

		\item Find another DVD drive, replace the built-in Toshiba drive with a new one and hope that it will work. I have not tested this suggestion, but it might work for you.

		\item Find the latest firmware upgrade 

	\end{itemize}

\end{itemize}

\end{document}
