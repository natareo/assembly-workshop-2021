---
title: "Arvo:  Vane Roles"
teaching: 20
exercises: 0
questions:
- "What services do Arvo vanes provide?"
- "How do I access system operations?"
objectives:
- "Understand architecture of the `%ames` network and communications."
- "Access file data from `%clay`."
- "Access arbitrary revisions of a file through `%clay`."
- "Manipulate remote data through `%eyre`/`%iris`."
- "Provide external commands to Arvo through `%khan`."
keypoints:
- "Vanes communicate by means of events."
- "`%ames` provides network communications including peer-to-peer events."
- "`%clay` instruments a typed version-controlled filesystem."
- "`%eyre` and `%iris` together offer client and server operations."
- "`%khan` is the external control plane."
---

##  Arvo Vanes

Each vane has a characteristic structure which identifies it as a vane to Arvo and allows it to handle moves consistently.  Most operations are one of three things:

1. A _scry_ or request for data (`++peek`).
2. An update (`++poke`).
3. An evaluation (of a core) (`++wish`).

In order to orient yourself around the kinds of things the Urbit OS does, it is worth a brief tour of the Arvo vanes.

### `%ames`, A Network

In a sense, `%ames` is the operative definition of an urbit on the network.  That is, from outside of one's own urbit, the only specification that must be hewed to is that `%ames` behaves a certain way in response to events. TODO

`%ames` implements a system expecting—and delivering—guaranteed one-time delivery.  This derives from an observation by \citeauthor{Yarvin2016}~in the Whitepaper:  "bus v. commands whatever"

UDP packet structure

network events
acks \& nacks

\subsection{Scrying into \ames}
\labsec{kr:a:scry}

`%ames` scry

\begin{table}[h!]
  \begin{center}
    \caption{`%ames` \dotket~Calls.}
    \label{ha:ames}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%x} & Get ship and peer information:  protocol version, peers, ship state, etc. & \\
    \end{tabular}
  \end{center}
\end{table}


\section[\behn]{\behn, A Timer}
\labsec{behn}

`%behn` is a simple vane that promises to emit events after—but never before—their timestamp.  This guarantee

As the shortest vane, we commend `%behn` to the student as an excellent subject for a first dive into the structure of a vane.

`%behn` maintains an event handler and a state.

Any task may have one of the following states:

\begin{lstlisting}
%born  born:event-core
%rest  (rest:event-core date=p.task)
%drip  (drip:event-core move=p.task)
%huck  (huck:event-core syn.task)
%trim  trim:event-core
%vega  vega:event-core
%wait  (wait:event-core date=p.task)
%wake  (wake:event-core error=~)
\end{lstlisting}

\subsection{Scrying into \behn}
\labsec{kr:b:scry}

\begin{table}[h!]
  \begin{center}
    \caption{`%behn` \dotket~Calls.}
    \label{ha:behn}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%x} & Get timers, timestamps, next timer to fire, etc. & \\
    \end{tabular}
  \end{center}
\end{table}

\section[`%clay` ]{`%clay` , A File System}
\labsec{clay}

https://github.com/davis68/martian-computing/blob/78dce6435f645f2135f09e228062a1371cf2ef9d/lessons/lesson22-clay-2.md

```
~tinnus-napbus
3:57 PM
what is the correct way to read a file on a remote ship? I've tried both warp and werp and I'm not getting a response, just messages in the target about clay something something indirect and then I get crash on fragment errors sometimes
with an %x care
~rovnys-ricfer
4:02 PM
warp should work, I think
make sure the file is actually there
i.e. can you do the same warp on the local ship?
```

\begin{itemize}
  \item  \texttt{aeon} is
  \item  \texttt{arch} is
  \item  \texttt{care} is the `%clay` ~submode, defined in \lull.
  \item  \texttt{desk} is
  \item  \texttt{dome} is
  mark cast file
\end{itemize}


\subsection{Data Model}
\labsec{kr:c:data}

paths

merges

marks

\subsection{Scrying into `%clay` }
\labsec{kr:c:scry}

`%clay` ~has the most sophisticated scry taxonomy.

\begin{table}[h!]
  \begin{center}
    \caption{`%clay` ~\dotket~Calls.}
    \label{ha:clay}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%a} & Expose file build into namespace. & \texttt{\dotket (vase \%ca /\~zod/home/1/lib/generators/hoon} \\
      \texttt{\%b} & Expose mark build into namespace. & \\
      \texttt{\%c} & Expose cast build into namespace. & \\
      \texttt{\%d} & diff; ask clay to validate .noun as .mark & \\
      \texttt{\%e} & & \\
      \texttt{\%f} & & \\
      \texttt{\%p} & Get the permissions that apply at path. & \\
      \texttt{\%r} & Get the data at a node (like \texttt{\%x}) and wrap it in a vase. & \\
      \texttt{\%s} & produce yaki or blob for given tako or lobe & \\
      \texttt{\%t} & produce the list of paths within a yaki with :pax as prefix & \\
      \texttt{\%u} & Check for existence of a node at aeon. & \\
      \texttt{\%v} & Get the desk state at a specified aeon. & \\
      \texttt{\%w} & Get all cases referring to the same revision as the given case. & \\
      \texttt{\%x} & Get the data at a node. & \\
      \texttt{\%y} & Get the \texttt{arch} (directory listing) at a node. & \\
      \texttt{\%z} & Get a recursive hash of a node and its children. & \\
    \end{tabular}
  \end{center}
\end{table}

% https://github.com/urbit/urbit/blob/master/pkg/arvo/sys/vane/gall.hoon#L1538
% https://urbit.org/blog/ford-fusion/


\subsection[`=ford` ]{`++ford` , A Build System}
\labsec{ford}

runes
%https://urbit.org/blog/ford-fusion/

\begin{table}[h!]
  \begin{center}
    \caption{`++ford` ~Runes.}
    \label{ha:ford}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{/-} & Import structure file from \texttt{sur}. & \\
      \texttt{/+} & Import library file from \texttt{lib}. & \\
      \texttt{/=} & Import user-specified file. & \\
      \texttt{/*} & Import contents of file converted by mark. & \\
    \end{tabular}
  \end{center}
\end{table}

importing with \texttt{*} is w/o face, foo=bar


\subsection[Marks]{Marks and conversions}
\labsec{kr:c:marks}

\subsection{Exercises}
\labsec{kr:c:exercises}

CSV Conversion Mark
Compose a mark capable of conversion from a CSV file to a plain-text file (and vice versa).

The ++grad arm can be copied from the hoon mark, since we are not concerned with preserving CSV integrity.

Conversion Tube
Use ++ford to produce a tube from hoon to txt.

Answer this question with the expression.


\section[`%dill` ]{`%dill` , A Terminal driver}
\labsec{dill}

\subsection{Scrying into `%dill` }
\labsec{kr:d:scry}

`%dill` ~scrys are unusual, in that they are typically only necessary for fine-grained Arvo control of the display.  Even command-line apps instrumented with \shoe~do not call into `%dill` ~commonly.  The only instance of use in the current Arvo kernel is in Herm, the terminal session manager.

\begin{table}[h!]
  \begin{center}
    \caption{`%dill` ~\dotket~Calls.}
    \label{ha:dill}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%x} & Get the current line or cursor position of default session. & \\
    \end{tabular}
  \end{center}
\end{table}


\section[`%eyre` ~\&~`%iris` ]{`%eyre` ~and `%iris` , Server and Client Vanes}
\labsec{eyre}

\subsection{Scrying into `%eyre` }
\labsec{kr:e:scry}

`%eyre` ~

\begin{table}[h!]
  \begin{center}
    \caption{`%jael` ~\dotket~Calls.}
    \label{ha:iris}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%x} & Get CORS etc TODO. & \\
      \texttt{\%\$} & Get . & \\  % TODO is this default subject?
    \end{tabular}
  \end{center}
\end{table}

\subsection{Scrying into `%iris` }
\labsec{kr:i:scry}

\begin{table}[h!]
  \begin{center}
    \caption{`%iris` ~\dotket~Calls.}
    \label{ha:iris}
    \begin{tabular}{lll}
      Symbol & Meaning & Example \\
      \hline \\
      \texttt{\%\$} & Get . & \\  % TODO is this default subject?
    \end{tabular}
  \end{center}
\end{table}

\section[`%jael` ]{`%jael` , Secretkeeper}
\labsec{jael}

\marginnote[2mm]{
`%jael`  is named after Jael, the wife of Heber, who kept mum and slew fleeing enemy general Sisera in Judges 4.  It also puns on “jail”.
}

`%jael` ~keeps secrets, the cryptographic keys that make it possible to securely control your Urbit.  Among other cryptographic facts, `%jael` ~keeps track of the following:

\begin{enumerate}
  \item  Subscription to \texttt{\%azimuth-tracker}, the current state of the Azimuth PKI.
  \item  Initial public and private keys for the ship.
  \item  Public keys of all galaxies.
  \item  Record of Ethereum registration for Azimuth.
\end{enumerate}

`%jael` ~weighs in as one of the shorter vanes, but is critical to Urbit as a secure network-first operating system.  `%jael` ~is in fact the first vane loaded after `%dill` ~when bootstrapping Arvo on a new instance.

\begin{lstlisting}

\end{lstlisting}


record the Ethereum block the public key is registered to,
record the URL of the Ethereum node used,
save the signature of the parent planet (if the ship is a moon),
load the initial public and private keys for the ship,
set the DNS suffix(es) used by the network (currently just arvo.network),
save the public keys of all galaxies,
set Jael to subscribe to %azimuth-tracker,
%slip a %init task to Ames, Clay, Gall, Dill, and Eyre, and %give an %init gift to Arvo, which then informs Unix that the initialization process has concluded.

~master-morzod
1:00 PM
the private key (ring) is 64 bytes, plus a tag byte
1:00
(met 3 sec:ex:(pit:nu:crub:crypto 512 eny))
65
the ship is up to 64 bits (or 128, if you want to support comet keypairs, which you could)
\begin{lstlisting}
=/  ,seed:jael  [who=`@p`0 lyf=1 sek=(bex 520) ~]  (met 3 (scot %uw (jam -)))
=/  ,seed:jael  [who=`@p`(bex 127) lyf=(bex 31) sek=(bex 520) ~]  (met 3 (scot %uw (jam -)))
\end{lstlisting}
likely lower and upper bounds


\href{https://urbit.org/docs/arvo/jael/jael-api/}{`%jael` ~API Reference}



\subsection{Scrying into `%jael` }
\labsec{kr:j:scry}




\marginnote[2mm]{
Sometimes residual elements of vane development leak into release code and yield insight into how the kernel developers produce new vanes.  For instance, when a new version of `%jael` ~was being developed, it was dubbed \texttt{kale} and used the scry path \texttt{\%k}.  This made it into a release version of \texttt{lib/ring} in Arvo 309 K.
}


{% include links.md %}
