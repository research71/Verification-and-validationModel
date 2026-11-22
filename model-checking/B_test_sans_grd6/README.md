# Detection test: guard removed (instance B)

Copy of `B_10aircraft_3scenarios` in which the guard `grd6` of the event `Start_takingoff` (machine `M4`) is removed:

    runway /: dom(lastlandingtime) or current_time >= lastlandingtime(runway) + RwySep

This copy is only used to show that the model checking detects a violation. It is not part of the proved model, and its proof obligations are not discharged.

`counterexample_inv17.prob2trace` replays the counterexample found by ProB (scenario 3, 47 events, 780 s): `AC1` lands, the landing terminates at 780 s, and `AC1` takes off at 780 s, so that `depTime(AC1) = lastlandingtime(RWY28L) = 780`, whereas the invariant `M4:inv17` requires `depTime >= lastlandingtime + RwySep` with `RwySep = 60`.

`detection_figure.png` combines two captures: (a) ProB reports the invariant violation (3.5 s, 259 states, 325 transitions) and (b) the evaluation of `M4:inv17` at the end of the trace, where `depTime(AC1) = 780` is below `lastlandingtime(RWY28L) + RwySep = 840`. The two captures are also given separately (`detection_a_model_checking.png`, `detection_b_inv17_evaluation.png`).

`detection_c_guards.png` shows the guards of `Start_takingoff` for `AC1` in the state before the violation (right column: all `true`, so the event is enabled at 780 s) and in the final state (left column); the condition on `lastlandingtime + RwySep` no longer appears among the guards.
