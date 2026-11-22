# Temporal properties validated with ProB

The twelve properties below are checked with ProB on the instance `B_10aircraft_3scenarios` (ten aircraft, horizon 3600 s, three traffic scenarios, urgency guards). All of them hold. Ids: SAF*i* is the safety property of critical case CS*i* (requirement R*i*), P is a possibility property, and L is a liveness property. The paper details SAF5 and L2; the other formulas are given here.

In the formulas, braces `{ }` delimit a predicate on the current state, written in ProB's ASCII syntax for Event-B. The identifiers are the variables and constants of the model, and `statusof(a)` takes the values `Blocked`, `ReadyL`, `Landing` and `Takingoff`. The formulas can be copied into ProB as they are (`ProB` > `LTL Model Checking` or `CTL Model Checking`).

| Id | Logic | Property | Result |
|---|---|---|---|
| SAF1 | LTL | Two aircraft that enter through the same point are separated by at least the minimum time. | holds |
| SAF2 | LTL | Two aircraft at the same merge point are separated by at least the minimum time. | holds |
| SAF3 | LTL | Two successive landings on the same runway are separated by at least the minimum time. | holds |
| SAF4 | LTL | The runway lights are consistent with the runway status: an unavailable runway has its lights on. | holds |
| SAF5 | LTL | A take-off and a landing on the same runway, in either order, are separated by at least the runway separation. | holds |
| SAF6 | LTL | Two successive take-offs are separated by at least the minimum time. | holds |
| P1 | CTL | An aircraft can be landing while another one is ready to land. | holds |
| P2 | CTL | A take-off can take place after a landing on the same runway. | holds |
| L1 | LTL | Eventually, every aircraft has entered the TMA and left it. | holds |
| L2 | CTL | On all paths, eventually every aircraft has entered the TMA and left it. | holds |
| L3 | LTL | Whenever an aircraft is blocked, eventually no aircraft is blocked. | holds |
| L4 | CTL | Whenever a runway is unavailable, on all paths all runways eventually become available again. | holds |

The capture of the results is in `results/ltl_ctl_results_B.png`.

## Safety (LTL)

### SAF1: separation at the TMA entry point (CS1, R1)
```
G {!a.(a:dom(tmaEntryTime) & a:dom(brandof) & a:dom(entryTMAOf) & entryTMAOf(a):dom(lastEntryTime) & entryTMAOf(a):dom(lastEntryBrand) & tmaEntryTime(a) < lastEntryTime(entryTMAOf(a)) => lastEntryTime(entryTMAOf(a)) - tmaEntryTime(a) >= Separation_time(brandof(a)|->lastEntryBrand(entryTMAOf(a))))}
```

### SAF2: separation at the merge point (CS2, R2)
```
G {!(a,b).(a:dom(mergeTime) & b:dom(mergeTime) & a/=b & a:dom(legOf) & b:dom(legOf) & a:dom(brandof) & b:dom(brandof) & PmsMerge(legPMS(legOf(a)))=PmsMerge(legPMS(legOf(b))) & mergeTime(a)<=mergeTime(b) => mergeTime(b)-mergeTime(a) >= Separation_time(brandof(a)|->brandof(b)))}
```

### SAF3: separation between two landings (CS3, R3)
```
G {!a.(a:aircrafts_in_TMA & statusof(a)=Landing & a:dom(clearanceL) & a:dom(TouchdownTime) & a:dom(brandof) & clearanceL(a):dom(lastlandingtime) & clearanceL(a):dom(lastlandingBrand) => TouchdownTime(a) >= lastlandingtime(clearanceL(a)) + Separation_time(lastlandingBrand(clearanceL(a))|->brandof(a)))}
```

### SAF4: runway status and lights (CS4, R4)
```
G {!r.(r:RUNWAY & r:dom(curr_RW_status) & r:dom(curr_RWL_status) => (curr_RW_status(r)=Unavailable => curr_RWL_status(r)=ON))}
```

### SAF5: separation between a landing and a take-off, in both orders (CS5, R5)
```
G ({!a.(a:aircrafts_in_TMA & statusof(a)=Takingoff & a:dom(depTime) & a:dom(depRwy) & depRwy(a):dom(lastlandingtime) => depTime(a) >= lastlandingtime(depRwy(a)) + RwySep)} & {!a.(a:aircrafts_in_TMA & statusof(a)=Landing & a:dom(clearanceL) & a:dom(TouchdownTime) & clearanceL(a):dom(lastDepTime) => TouchdownTime(a) >= lastDepTime(clearanceL(a)) + RwySep)})
```

### SAF6: separation between two take-offs (CS6, R6)
```
G {!a.(a:aircrafts_in_TMA & statusof(a)=Takingoff & a:dom(depTime) & a:dom(depRwy) & a:dom(brandof) & depRwy(a):dom(lastDepTime) & depRwy(a):dom(lastDepBrand) => depTime(a) >= lastDepTime(depRwy(a)) + DepDepSep(lastDepBrand(depRwy(a))|->brandof(a)))}
```

## Possibility (CTL)

### P1: a landing while another aircraft is ready to land
```
EF {#(a,b).(a:aircrafts_in_TMA & b:aircrafts_in_TMA & a/=b & statusof(a)=Landing & statusof(b)=ReadyL)}
```

### P2: a take-off after a landing on the same runway
```
EF {#a.(a:aircrafts_in_TMA & statusof(a)=Takingoff & a:dom(depRwy) & depRwy(a):dom(lastlandingtime))}
```

## Liveness

### L1 (LTL): every aircraft eventually enters and leaves
```
F {!a.(a:AIRCRAFTS => a:dom(brandof) & a/:aircrafts_in_TMA)}
```

### L2 (CTL): on all paths, every aircraft eventually enters and leaves
```
AF {!a.(a:AIRCRAFTS => a:dom(brandof) & a/:aircrafts_in_TMA)}
```

### L3 (LTL): a blocked aircraft is eventually released
```
G ({#a.(a:aircrafts_in_TMA & statusof(a)=Blocked)} => F {!a.(a:aircrafts_in_TMA => statusof(a)/=Blocked)})
```

### L4 (CTL): an unavailable runway eventually becomes available again
```
AG ({#r.(r:RUNWAY & curr_RW_status(r)=Unavailable)} => AF {!r.(r:RUNWAY => curr_RW_status(r)=Available)})
```

L1 to L4 hold under the urgency guards of instance B. Without them, the path in which no aircraft acts until the horizon falsifies L1 and L2.
