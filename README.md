# Circuit Descriptions

## General

### `constants.cir`

Various constants for use in SFQ Circuits.
- `Phi0`: the flux quanta, $\Phi_0$
- `qe`: the fundamental electric charge, $e$
- `hbar`: the reduced Planck's constant, $\hbar$

### `junction.cir`

Junction models.
- `rcsj_model`: the "Resistively and Capacitively Shunted Junction" (RCSJ) model. 
    - `IC`: the *ideal* junction critical current, $I_c$.
    - `R`: the value of the shunt resistor, $R$, in Ohms.
    - `beta_c`: the junction dampening parameter, $\beta_c$. $\beta_c = 1$ corresponds to a *critically damped* junction; $\beta_c > 1$ is *underdamped*, $\beta_c < 1$ *overdamped*.
    - Within the model, the $\beta_c$ and $R$ values are used to calculate the appropriate shunt capacitance value.
    - *Node*: `pos`: positive node; `neg`: negative node; `phase`: phase node

- `sj`: a "shunted junction" model; an overdamped version
    - `R`: the shunt resistance value
    - `IC`: the junction critical current
    - *Node*: `p`: positive node; `n`: negative node; `ph`: phase node

- `cdjj`: a "critically damped jj" model
    - `IC`: the junction critical current
    - internally, $\beta_c$ is set to 1.
    - *Node*: `p`: positive node; `n`: negative node; `ph`: phase node

### `misc.cir`

Miscellaneous, general circuit models.
- `ibias`: a bias current source ramped from zero to avoid some transient effects.
    - `IB`: value of the bias current, $I_b$; in Amps.
    - `t`: the rise time, from 0 bias current to $I_b$. Default, 100ps.
    - *Nodes*: `p`: positive node; `n`: negative node

## Rapid Single Flux Quantum (RSFQ)

For these subcircuits to function, include the file `./general.cir` to add junction model, etc.

### `jtl.cir`

Josephson transmission line (JTL) circuit model.
- `jtlmod`: a JTL "module". 
    - `IC`: the critical current of all four internal junctions, $I_c$.
    - *Nodes*: `1`: input, `4` output; (however, the circuit is symmetric).

### `buffer.cir`

- `buffer`: simple buffer circuit
    - `IC`: the critical current of the *smaller* junction, $I_{c2}$; $I_{c1}$ is scaled accordingly
    - *Nodes*: `A`: input, `B`: output

### `dc-sfq_converter.cir`

- `dc_to_sfq`: DC/SFQ converter
    - `IC`: critical current of all junctions, $I_c$; default is .125mA.
    - `rise_time`: rise time of internal bias currents; see `ibias`.
    - *Nodes*: `i`: input (for current); `o`: output (where SFQ is emitted)

- `sfq_mod`: an "SFQ Module"
    - `IC`: critical current of junctions within the DC/SFQ converter
    - `freq`: frequency; intended for use with a flux shuttle module (see below); used for timing.
    - `imax`: maximum input current value.
    - `delay`: number of *cycles* (in terms of the above frequency) before the input current begins to rise
    - Period, $T=1/f$ is calculated; delay-period product is calculated. Rise time is half a period; a PWL (piecewise linear) function creates a "triangle" wave, starting at delay and taking a full period, from beginning to end. 
    - *Nodes*: `o`: output (where SFQ is emitted)

### `compound_buffers.cir`

- `swbuffer`: a "sandwich buffer"
    - `IC`: the critical current of both the `jtlmod` and `buffer`
    - *Nodes*: `A`: input; `B`: output

### `pulse_splitter.cir`

- `pulse_splitter`: an RSFQ pulse splitter (one input to two identical outputs); see [1991LikharevRSFQ], Fig 4(a).
    - `IC`: the critical current of the *smaller* junctions ($I_{c2}=I_{c3}$)
    - *Nodes*: `A`: input; `B`,`C`: outputs.
        - these are somewhat symmetric; however, the bias and critical current of junction 1 is bigger than 2,3.
    
### `confluence_buffer.cir`

- `confluence_buffer`: an RSFQ confluence buffer (two inputs into 1); see [1991LikharevRSFQ] Fig. 6(a).
    - `IC`: the critical current of the *smaller* junctions ($I_{c3}=I_{c4}=I_{c5}$)
    - *Nodes*: `A`,`B`: inputs; `C`: output. *Not* symmetric; simple buffer behavior is "built in"

### `rs_flip-flop.cir`

- `ff`: basic RS flip-flop; see [1991LikharevRSFQ] Fig. 7(a).
    - `IC`: the critical current of the *smaller* junctions; $I_c=I_{c1}=I_{c2}$.
    - *Nodes*: `R`: "reset"; `S`: "set"; `F`: output

### `t_flip-flop.cir`

- `tff`: a T flip-flop; see [1991LikharevRSFQ] Fig. 8(a).
    - `IC`: the critical current of the *smaller* junctions; $I_c=I_{c2}=I_{c4}$
    - *Nodes*: `A`: input; `F0`,`F1`: alternating outputs

### `or_cell.cir`

- `or`: an RSFQ OR cell; see [1991LikharevRSFQ] Fig. 10(a).
    - `IC`: the critical current of the smaller junctions
    - *Nodes*: `A`,`B`: inputs; `F`: output; `T`: clock

### `inverter.cir`

- `not`: an RSFQ inverter; modified from many RSFQ resources
    - `IC`: critical current of attached *JTL* junctions; the internal junctions have a 40% larger critical current
    - *Nodes*: `A`: input; `T`: clock; `F`: output

- `not_with_swb`: encapsulates above not with above `swbuffer`s
    - `IC`: critical current of JTL junctions
    - *Nodes*: `A`: input; `T`: clock; `F`: output

### `and_or_cell.cir`

- `and_or`: an RSFQ AND-OR cell.
    - `IC`: critical current of the smallest junctions
    - *Nodes*: `A`,`B`,`C`,`D`: inputs (symmetric); `F`: output; `T`: clock

### `shift_register.cir`
- `sr_mod`: "shift register module"
    - `IC`: critical current of the smallest (middle) junctions
    - *Nodes*: `S`: data in (bottom left); `So`: data out (bottom right); `To`: clock out (top left); `T`: clock in (top right)


## AC Biased Shift Register

### `fluxshuttle.cir`

- `fs_mod`: flux shuttle module
    - `freq`: the frequency of the internal AC clock
    - `IC`: the critical current of the four junctions
    - `imax`: the scale factor on `IC`; determines the magnitude of the AC clock in terms of $I_c$.
    - *Nodes*: `A`: input; `B`: output; these are *not* symmetric, due to inductances

### `fluxshuttle_corner.cir`

- `fs_corner_mod`: flux shuttle corner module
    - `freq`: the frequency of the internal AC clock
    - `IC`: the critical current of the four large junctions
    - `IC_out`: the critical current of the output junction
    - `imax`: the scale factor on `IC`; determines the magnitude of the AC clock in terms of $I_c$.
    - `lout_fact`: a scale factor on $\Phi_0/I_{c,out}$.
    - *Nodes*: `A`: input; `B`: output; these are *not* symmetric, due to inductances

