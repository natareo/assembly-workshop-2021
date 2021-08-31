---
title: "Hoon Basics:  Advanced Data Structures"
teaching: 30
exercises: 15
questions:
- "How are structured data typically represented, populated, and accessed in Hoon?"
- "Which common patterns in Hoon exist?"
objectives:
- "Identify common Hoon molds:  units, sets, maps, jars, and jugs."
- "Explore the standard library’s functionality:  text parsing & processing, functional hacks, randomness, hashing, time, and so forth."
keypoints:
- "Doors generalize the notion of a gate to a gate-factory."
- "Units, sets, maps, jars, and jugs provide a principled way to access and operate on data."
---

##  Data Structures
\labsec{he:data-structures}

\subsubsection{Units}
\labsec{he:unit}

Every atom in Hoon is an unsigned integer, even if interpreted by an aura.  Auras do not carry prescriptive power, however, so they cannot be used to fundamentally distinguish a \texttt{NULL}-style or \texttt{NaN}-style non-result.  That is, if one receives a \sig~back from a query, how does one distinguish a value of zero from a non-result (missing value)?

Units mitigate this situation by acting as a type union of \sig~(for no result) and a cell \texttt{[\textasciitilde~u=item]} containing the returned item with face \texttt{u}.

\begin{lstlisting}[style=nonumbers]
++  unit
  |$  [item]
  $@(~ [~ u=item])
\end{lstlisting}


\subsubsection{Maps}
\labsec{he:map}

\subsubsection{Sets}
\labsec{he:set}

\subsubsection{Trees}
\labsec{he:tree}

\subsubsection{Dimes}
\labsec{he:dime}

jars, jugs  OPtional


{% include links.md %}
