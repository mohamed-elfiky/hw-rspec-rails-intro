## Part 6: the final spec

There's just one example left to write, to check that the TMDb search
results will be made available to the response view.  
The RSpec `assigns()` method keeps track of what instance variables were
assigned in the controller method.  Hence \C{assigns(:movies)} returns
whatever value (if any) was assigned to \C{@movies} in \ctrlmeth, and 
our spec just has to verify that the controller action correctly sets
up this variable.
In our case, we've already arranged to return our doubles as the
result of the faked-out method call, so the correct behavior for
\ctrlmeth\ would be to set
\C{@movies} to this value, as
line 21 of Figure~\ref{fig:assigns} asserts.

\index{search\_tmdb!specs|textit}
\index{search\_tmdb!code sample|textit}
\codefilefigure[0311b5b4bbb9f17060e7e0e2210ff6ab]{ch_tdd/code/spec/movies_controller_spec.6.rb}%
  {fig:assigns}%
  {Asserting that \C{@movie} is set up correctly by \ctrlmeth.  Lines
    18--22 in this listing replace line 18 in Figure~\ref{fig:dry_specs}.}

\begin{elaboration}{More than we need?}
  Strictly speaking, for the purposes of this example the stubbed
  \index{find\_in\_tmdb!usage|textit}
  \modmeth\ could have returned any value at all, such as the string ``I am a
  movie'', because the \emph{only} behavior tested by this example
  is whether the correct instance variable is being set up and made
  available to the view.  In particular, this example doesn't care what
  the \emph{value} of that variable is, or whether \modmeth\ is returning
  something sensible.  But since we already had doubles set up, it was
  easy enough to use them in this example.
\end{elaboration}
