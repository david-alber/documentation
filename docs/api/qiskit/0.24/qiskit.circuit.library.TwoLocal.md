<span id="qiskit-circuit-library-twolocal" />

# qiskit.circuit.library.TwoLocal

<span id="undefined" />

`TwoLocal(num_qubits=None, rotation_blocks=None, entanglement_blocks=None, entanglement='full', reps=3, skip_unentangled_qubits=False, skip_final_rotation_layer=False, parameter_prefix='θ', insert_barriers=False, initial_state=None)`

The two-local circuit.

The two-local circuit is a parameterized circuit consisting of alternating rotation layers and entanglement layers. The rotation layers are single qubit gates applied on all qubits. The entanglement layer uses two-qubit gates to entangle the qubits according to a strategy set using `entanglement`. Both the rotation and entanglement gates can be specified as string (e.g. `'ry'` or `'cx'`), as gate-type (e.g. `RYGate` or `CXGate`) or as QuantumCircuit (e.g. a 1-qubit circuit or 2-qubit circuit).

A set of default entanglement strategies is provided:

*   `'full'` entanglement is each qubit is entangled with all the others.
*   `'linear'` entanglement is qubit $i$ entangled with qubit $i + 1$, for all $i \in \{0, 1, ... , n - 2\}$, where $n$ is the total number of qubits.
*   `'circular'` entanglement is linear entanglement but with an additional entanglement of the first and last qubit before the linear part.
*   `'sca'` (shifted-circular-alternating) entanglement is a generalized and modified version of the proposed circuit 14 in [Sim et al.](https://arxiv.org/abs/1905.10876). It consists of circular entanglement where the ‘long’ entanglement connecting the first with the last qubit is shifted by one each block. Furthermore the role of control and target qubits are swapped every block (therefore alternating).

The entanglement can further be specified using an entangler map, which is a list of index pairs, such as

```python
>>> entangler_map = [(0, 1), (1, 2), (2, 0)]
```

If different entanglements per block should be used, provide a list of entangler maps. See the examples below on how this can be used.

```python
>>> entanglement = [entangler_map_layer_1, entangler_map_layer_2, ... ]
```

Barriers can be inserted in between the different layers for better visualization using the `insert_barriers` attribute.

For each parameterized gate a new parameter is generated using a `ParameterVector`. The name of these parameters can be chosen using the `parameter_prefix`.

## Examples

```python
>>> two = TwoLocal(3, 'ry', 'cx', 'linear', reps=2, insert_barriers=True)
>>> print(two)  # decompose the layers into standard gates
     ┌──────────┐ ░            ░ ┌──────────┐ ░            ░ ┌──────────┐
q_0: ┤ Ry(θ[0]) ├─░───■────────░─┤ Ry(θ[3]) ├─░───■────────░─┤ Ry(θ[6]) ├
     ├──────────┤ ░ ┌─┴─┐      ░ ├──────────┤ ░ ┌─┴─┐      ░ ├──────────┤
q_1: ┤ Ry(θ[1]) ├─░─┤ X ├──■───░─┤ Ry(θ[4]) ├─░─┤ X ├──■───░─┤ Ry(θ[7]) ├
     ├──────────┤ ░ └───┘┌─┴─┐ ░ ├──────────┤ ░ └───┘┌─┴─┐ ░ ├──────────┤
q_2: ┤ Ry(θ[2]) ├─░──────┤ X ├─░─┤ Ry(θ[5]) ├─░──────┤ X ├─░─┤ Ry(θ[8]) ├
     └──────────┘ ░      └───┘ ░ └──────────┘ ░      └───┘ ░ └──────────┘
```

```python
>>> two = TwoLocal(3, ['ry','rz'], 'cz', 'full', reps=1, insert_barriers=True)
>>> qc = QuantumCircuit(3)
>>> qc += two
>>> print(qc.decompose().draw())
     ┌──────────┐┌──────────┐ ░           ░ ┌──────────┐ ┌──────────┐
q_0: ┤ Ry(θ[0]) ├┤ Rz(θ[3]) ├─░──■──■─────░─┤ Ry(θ[6]) ├─┤ Rz(θ[9]) ├
     ├──────────┤├──────────┤ ░  │  │     ░ ├──────────┤┌┴──────────┤
q_1: ┤ Ry(θ[1]) ├┤ Rz(θ[4]) ├─░──■──┼──■──░─┤ Ry(θ[7]) ├┤ Rz(θ[10]) ├
     ├──────────┤├──────────┤ ░     │  │  ░ ├──────────┤├───────────┤
q_2: ┤ Ry(θ[2]) ├┤ Rz(θ[5]) ├─░─────■──■──░─┤ Ry(θ[8]) ├┤ Rz(θ[11]) ├
     └──────────┘└──────────┘ ░           ░ └──────────┘└───────────┘
```

```python
>>> entangler_map = [[0, 1], [1, 2], [2, 0]]  # circular entanglement for 3 qubits
>>> two = TwoLocal(3, 'x', 'crx', entangler_map, reps=1)
>>> print(two)  # note: no barriers inserted this time!
        ┌───┐                             ┌──────────┐┌───┐
q_0: |0>┤ X ├─────■───────────────────────┤ Rx(θ[2]) ├┤ X ├
        ├───┤┌────┴─────┐            ┌───┐└─────┬────┘└───┘
q_1: |0>┤ X ├┤ Rx(θ[0]) ├─────■──────┤ X ├──────┼──────────
        ├───┤└──────────┘┌────┴─────┐└───┘      │     ┌───┐
q_2: |0>┤ X ├────────────┤ Rx(θ[1]) ├───────────■─────┤ X ├
        └───┘            └──────────┘                 └───┘
```

```python
>>> entangler_map = [[0, 3], [0, 2]]  # entangle the first and last two-way
>>> two = TwoLocal(4, [], 'cry', entangler_map, reps=1)
>>> circuit = two + two
>>> print(circuit.decompose().draw())  # note, that the parameters are the same!
q_0: ─────■───────────■───────────■───────────■──────
          │           │           │           │
q_1: ─────┼───────────┼───────────┼───────────┼──────
          │      ┌────┴─────┐     │      ┌────┴─────┐
q_2: ─────┼──────┤ Ry(θ[1]) ├─────┼──────┤ Ry(θ[1]) ├
     ┌────┴─────┐└──────────┘┌────┴─────┐└──────────┘
q_3: ┤ Ry(θ[0]) ├────────────┤ Ry(θ[0]) ├────────────
     └──────────┘            └─────────
```

```python
>>> layer_1 = [(0, 1), (0, 2)]
>>> layer_2 = [(1, 2)]
>>> two = TwoLocal(3, 'x', 'cx', [layer_1, layer_2], reps=2, insert_barriers=True)
>>> print(two)
     ┌───┐ ░            ░ ┌───┐ ░       ░ ┌───┐
q_0: ┤ X ├─░───■────■───░─┤ X ├─░───────░─┤ X ├
     ├───┤ ░ ┌─┴─┐  │   ░ ├───┤ ░       ░ ├───┤
q_1: ┤ X ├─░─┤ X ├──┼───░─┤ X ├─░───■───░─┤ X ├
     ├───┤ ░ └───┘┌─┴─┐ ░ ├───┤ ░ ┌─┴─┐ ░ ├───┤
q_2: ┤ X ├─░──────┤ X ├─░─┤ X ├─░─┤ X ├─░─┤ X ├
     └───┘ ░      └───┘ ░ └───┘ ░ └───┘ ░ └───┘
```

Construct a new two-local circuit.

**Parameters**

*   **num\_qubits** (`Optional`\[`int`]) – The number of qubits of the two-local circuit.
*   **rotation\_blocks** (`Union`\[`str`, `List`\[`str`], `type`, `List`\[`type`], `QuantumCircuit`, `List`\[`QuantumCircuit`], `None`]) – The gates used in the rotation layer. Can be specified via the name of a gate (e.g. ‘ry’) or the gate type itself (e.g. RYGate). If only one gate is provided, the gate same gate is applied to each qubit. If a list of gates is provided, all gates are applied to each qubit in the provided order. See the Examples section for more detail.
*   **entanglement\_blocks** (`Union`\[`str`, `List`\[`str`], `type`, `List`\[`type`], `QuantumCircuit`, `List`\[`QuantumCircuit`], `None`]) – The gates used in the entanglement layer. Can be specified in the same format as rotation\_blocks.
*   **entanglement** (`Union`\[`str`, `List`\[`List`\[`int`]], `Callable`\[\[`int`], `List`\[`int`]]]) – Specifies the entanglement structure. Can be a string (‘full’, ‘linear’ , ‘circular’ or ‘sca’), a list of integer-pairs specifying the indices of qubits entangled with one another, or a callable returning such a list provided with the index of the entanglement layer. Default to ‘full’ entanglement. See the Examples section for more detail.
*   **reps** (`int`) – Specifies how often a block consisting of a rotation layer and entanglement layer is repeated.
*   **skip\_unentangled\_qubits** (`bool`) – If True, the single qubit gates are only applied to qubits that are entangled with another qubit. If False, the single qubit gates are applied to each qubit in the Ansatz. Defaults to False.
*   **skip\_final\_rotation\_layer** (`bool`) – If False, a rotation layer is added at the end of the ansatz. If True, no rotation layer is added.
*   **parameter\_prefix** (`str`) – The parameterized gates require a parameter to be defined, for which we use instances of qiskit.circuit.Parameter. The name of each parameter will be this specified prefix plus its index.
*   **insert\_barriers** (`bool`) – If True, barriers are inserted in between each layer. If False, no barriers are inserted. Defaults to False.
*   **initial\_state** (`Optional`\[`Any`]) – An InitialState object to prepend to the circuit.

<span id="undefined" />

`__init__(num_qubits=None, rotation_blocks=None, entanglement_blocks=None, entanglement='full', reps=3, skip_unentangled_qubits=False, skip_final_rotation_layer=False, parameter_prefix='θ', insert_barriers=False, initial_state=None)`

Construct a new two-local circuit.

**Parameters**

*   **num\_qubits** (`Optional`\[`int`]) – The number of qubits of the two-local circuit.
*   **rotation\_blocks** (`Union`\[`str`, `List`\[`str`], `type`, `List`\[`type`], `QuantumCircuit`, `List`\[`QuantumCircuit`], `None`]) – The gates used in the rotation layer. Can be specified via the name of a gate (e.g. ‘ry’) or the gate type itself (e.g. RYGate). If only one gate is provided, the gate same gate is applied to each qubit. If a list of gates is provided, all gates are applied to each qubit in the provided order. See the Examples section for more detail.
*   **entanglement\_blocks** (`Union`\[`str`, `List`\[`str`], `type`, `List`\[`type`], `QuantumCircuit`, `List`\[`QuantumCircuit`], `None`]) – The gates used in the entanglement layer. Can be specified in the same format as rotation\_blocks.
*   **entanglement** (`Union`\[`str`, `List`\[`List`\[`int`]], `Callable`\[\[`int`], `List`\[`int`]]]) – Specifies the entanglement structure. Can be a string (‘full’, ‘linear’ , ‘circular’ or ‘sca’), a list of integer-pairs specifying the indices of qubits entangled with one another, or a callable returning such a list provided with the index of the entanglement layer. Default to ‘full’ entanglement. See the Examples section for more detail.
*   **reps** (`int`) – Specifies how often a block consisting of a rotation layer and entanglement layer is repeated.
*   **skip\_unentangled\_qubits** (`bool`) – If True, the single qubit gates are only applied to qubits that are entangled with another qubit. If False, the single qubit gates are applied to each qubit in the Ansatz. Defaults to False.
*   **skip\_final\_rotation\_layer** (`bool`) – If False, a rotation layer is added at the end of the ansatz. If True, no rotation layer is added.
*   **parameter\_prefix** (`str`) – The parameterized gates require a parameter to be defined, for which we use instances of qiskit.circuit.Parameter. The name of each parameter will be this specified prefix plus its index.
*   **insert\_barriers** (`bool`) – If True, barriers are inserted in between each layer. If False, no barriers are inserted. Defaults to False.
*   **initial\_state** (`Optional`\[`Any`]) – An InitialState object to prepend to the circuit.

## Methods

|                                                                                                                                                                             |                                                                                                                                     |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`__init__`](#qiskit.circuit.library.TwoLocal.__init__ "qiskit.circuit.library.TwoLocal.__init__")(\[num\_qubits, rotation\_blocks, …])                                     | Construct a new two-local circuit.                                                                                                  |
| [`add_calibration`](#qiskit.circuit.library.TwoLocal.add_calibration "qiskit.circuit.library.TwoLocal.add_calibration")(gate, qubits, schedule\[, params])                  | Register a low-level, custom pulse definition for the given gate.                                                                   |
| [`add_layer`](#qiskit.circuit.library.TwoLocal.add_layer "qiskit.circuit.library.TwoLocal.add_layer")(other\[, entanglement, front])                                        | Append another layer to the NLocal.                                                                                                 |
| [`add_register`](#qiskit.circuit.library.TwoLocal.add_register "qiskit.circuit.library.TwoLocal.add_register")(\*regs)                                                      | Add registers.                                                                                                                      |
| [`append`](#qiskit.circuit.library.TwoLocal.append "qiskit.circuit.library.TwoLocal.append")(instruction\[, qargs, cargs])                                                  | Append one or more instructions to the end of the circuit, modifying the circuit in place.                                          |
| [`assign_parameters`](#qiskit.circuit.library.TwoLocal.assign_parameters "qiskit.circuit.library.TwoLocal.assign_parameters")(param\_dict\[, inplace])                      | Assign parameters to the n-local circuit.                                                                                           |
| [`barrier`](#qiskit.circuit.library.TwoLocal.barrier "qiskit.circuit.library.TwoLocal.barrier")(\*qargs)                                                                    | Apply `Barrier`.                                                                                                                    |
| [`bind_parameters`](#qiskit.circuit.library.TwoLocal.bind_parameters "qiskit.circuit.library.TwoLocal.bind_parameters")(value\_dict)                                        | Assign numeric parameters to values yielding a new circuit.                                                                         |
| [`cast`](#qiskit.circuit.library.TwoLocal.cast "qiskit.circuit.library.TwoLocal.cast")(value, \_type)                                                                       | Best effort to cast value to type.                                                                                                  |
| [`cbit_argument_conversion`](#qiskit.circuit.library.TwoLocal.cbit_argument_conversion "qiskit.circuit.library.TwoLocal.cbit_argument_conversion")(clbit\_representation)   | Converts several classical bit representations (such as indexes, range, etc.) into a list of classical bits.                        |
| [`ccx`](#qiskit.circuit.library.TwoLocal.ccx "qiskit.circuit.library.TwoLocal.ccx")(control\_qubit1, control\_qubit2, target\_qubit)                                        | Apply [`CCXGate`](qiskit.circuit.library.CCXGate#qiskit.circuit.library.CCXGate "qiskit.circuit.library.CCXGate").                  |
| [`ch`](#qiskit.circuit.library.TwoLocal.ch "qiskit.circuit.library.TwoLocal.ch")(control\_qubit, target\_qubit\[, label, …])                                                | Apply [`CHGate`](qiskit.circuit.library.CHGate#qiskit.circuit.library.CHGate "qiskit.circuit.library.CHGate").                      |
| [`cls_instances`](#qiskit.circuit.library.TwoLocal.cls_instances "qiskit.circuit.library.TwoLocal.cls_instances")()                                                         | Return the current number of instances of this class, useful for auto naming.                                                       |
| [`cls_prefix`](#qiskit.circuit.library.TwoLocal.cls_prefix "qiskit.circuit.library.TwoLocal.cls_prefix")()                                                                  | Return the prefix to use for auto naming.                                                                                           |
| [`cnot`](#qiskit.circuit.library.TwoLocal.cnot "qiskit.circuit.library.TwoLocal.cnot")(control\_qubit, target\_qubit\[, label, …])                                          | Apply [`CXGate`](qiskit.circuit.library.CXGate#qiskit.circuit.library.CXGate "qiskit.circuit.library.CXGate").                      |
| [`combine`](#qiskit.circuit.library.TwoLocal.combine "qiskit.circuit.library.TwoLocal.combine")(rhs)                                                                        | Append rhs to self if self contains compatible registers.                                                                           |
| [`compose`](#qiskit.circuit.library.TwoLocal.compose "qiskit.circuit.library.TwoLocal.compose")(other\[, qubits, clbits, front, inplace])                                   | Compose circuit with `other` circuit or instruction, optionally permuting wires.                                                    |
| [`control`](#qiskit.circuit.library.TwoLocal.control "qiskit.circuit.library.TwoLocal.control")(\[num\_ctrl\_qubits, label, ctrl\_state])                                   | Control this circuit on `num_ctrl_qubits` qubits.                                                                                   |
| [`copy`](#qiskit.circuit.library.TwoLocal.copy "qiskit.circuit.library.TwoLocal.copy")(\[name])                                                                             | Copy the circuit.                                                                                                                   |
| [`count_ops`](#qiskit.circuit.library.TwoLocal.count_ops "qiskit.circuit.library.TwoLocal.count_ops")()                                                                     | Count each operation kind in the circuit.                                                                                           |
| [`cp`](#qiskit.circuit.library.TwoLocal.cp "qiskit.circuit.library.TwoLocal.cp")(theta, control\_qubit, target\_qubit\[, …])                                                | Apply [`CPhaseGate`](qiskit.circuit.library.CPhaseGate#qiskit.circuit.library.CPhaseGate "qiskit.circuit.library.CPhaseGate").      |
| [`crx`](#qiskit.circuit.library.TwoLocal.crx "qiskit.circuit.library.TwoLocal.crx")(theta, control\_qubit, target\_qubit\[, …])                                             | Apply [`CRXGate`](qiskit.circuit.library.CRXGate#qiskit.circuit.library.CRXGate "qiskit.circuit.library.CRXGate").                  |
| [`cry`](#qiskit.circuit.library.TwoLocal.cry "qiskit.circuit.library.TwoLocal.cry")(theta, control\_qubit, target\_qubit\[, …])                                             | Apply [`CRYGate`](qiskit.circuit.library.CRYGate#qiskit.circuit.library.CRYGate "qiskit.circuit.library.CRYGate").                  |
| [`crz`](#qiskit.circuit.library.TwoLocal.crz "qiskit.circuit.library.TwoLocal.crz")(theta, control\_qubit, target\_qubit\[, …])                                             | Apply [`CRZGate`](qiskit.circuit.library.CRZGate#qiskit.circuit.library.CRZGate "qiskit.circuit.library.CRZGate").                  |
| [`cswap`](#qiskit.circuit.library.TwoLocal.cswap "qiskit.circuit.library.TwoLocal.cswap")(control\_qubit, target\_qubit1, …\[, …])                                          | Apply [`CSwapGate`](qiskit.circuit.library.CSwapGate#qiskit.circuit.library.CSwapGate "qiskit.circuit.library.CSwapGate").          |
| [`csx`](#qiskit.circuit.library.TwoLocal.csx "qiskit.circuit.library.TwoLocal.csx")(control\_qubit, target\_qubit\[, label, …])                                             | Apply [`CSXGate`](qiskit.circuit.library.CSXGate#qiskit.circuit.library.CSXGate "qiskit.circuit.library.CSXGate").                  |
| [`cu`](#qiskit.circuit.library.TwoLocal.cu "qiskit.circuit.library.TwoLocal.cu")(theta, phi, lam, gamma, control\_qubit, …)                                                 | Apply [`CUGate`](qiskit.circuit.library.CUGate#qiskit.circuit.library.CUGate "qiskit.circuit.library.CUGate").                      |
| [`cu1`](#qiskit.circuit.library.TwoLocal.cu1 "qiskit.circuit.library.TwoLocal.cu1")(theta, control\_qubit, target\_qubit\[, …])                                             | Apply [`CU1Gate`](qiskit.circuit.library.CU1Gate#qiskit.circuit.library.CU1Gate "qiskit.circuit.library.CU1Gate").                  |
| [`cu3`](#qiskit.circuit.library.TwoLocal.cu3 "qiskit.circuit.library.TwoLocal.cu3")(theta, phi, lam, control\_qubit, target\_qubit)                                         | Apply [`CU3Gate`](qiskit.circuit.library.CU3Gate#qiskit.circuit.library.CU3Gate "qiskit.circuit.library.CU3Gate").                  |
| [`cx`](#qiskit.circuit.library.TwoLocal.cx "qiskit.circuit.library.TwoLocal.cx")(control\_qubit, target\_qubit\[, label, …])                                                | Apply [`CXGate`](qiskit.circuit.library.CXGate#qiskit.circuit.library.CXGate "qiskit.circuit.library.CXGate").                      |
| [`cy`](#qiskit.circuit.library.TwoLocal.cy "qiskit.circuit.library.TwoLocal.cy")(control\_qubit, target\_qubit\[, label, …])                                                | Apply [`CYGate`](qiskit.circuit.library.CYGate#qiskit.circuit.library.CYGate "qiskit.circuit.library.CYGate").                      |
| [`cz`](#qiskit.circuit.library.TwoLocal.cz "qiskit.circuit.library.TwoLocal.cz")(control\_qubit, target\_qubit\[, label, …])                                                | Apply [`CZGate`](qiskit.circuit.library.CZGate#qiskit.circuit.library.CZGate "qiskit.circuit.library.CZGate").                      |
| [`dcx`](#qiskit.circuit.library.TwoLocal.dcx "qiskit.circuit.library.TwoLocal.dcx")(qubit1, qubit2)                                                                         | Apply [`DCXGate`](qiskit.circuit.library.DCXGate#qiskit.circuit.library.DCXGate "qiskit.circuit.library.DCXGate").                  |
| [`decompose`](#qiskit.circuit.library.TwoLocal.decompose "qiskit.circuit.library.TwoLocal.decompose")()                                                                     | Call a decomposition pass on this circuit, to decompose one level (shallow decompose).                                              |
| [`delay`](#qiskit.circuit.library.TwoLocal.delay "qiskit.circuit.library.TwoLocal.delay")(duration\[, qarg, unit])                                                          | Apply [`Delay`](qiskit.circuit.Delay#qiskit.circuit.Delay "qiskit.circuit.Delay").                                                  |
| [`depth`](#qiskit.circuit.library.TwoLocal.depth "qiskit.circuit.library.TwoLocal.depth")()                                                                                 | Return circuit depth (i.e., length of critical path).                                                                               |
| [`diag_gate`](#qiskit.circuit.library.TwoLocal.diag_gate "qiskit.circuit.library.TwoLocal.diag_gate")(diag, qubit)                                                          | Deprecated version of QuantumCircuit.diagonal.                                                                                      |
| [`diagonal`](#qiskit.circuit.library.TwoLocal.diagonal "qiskit.circuit.library.TwoLocal.diagonal")(diag, qubit)                                                             | Attach a diagonal gate to a circuit.                                                                                                |
| [`draw`](#qiskit.circuit.library.TwoLocal.draw "qiskit.circuit.library.TwoLocal.draw")(\[output, scale, filename, style, …])                                                | Draw the quantum circuit.                                                                                                           |
| [`extend`](#qiskit.circuit.library.TwoLocal.extend "qiskit.circuit.library.TwoLocal.extend")(rhs)                                                                           | Append QuantumCircuit to the right hand side if it contains compatible registers.                                                   |
| [`fredkin`](#qiskit.circuit.library.TwoLocal.fredkin "qiskit.circuit.library.TwoLocal.fredkin")(control\_qubit, target\_qubit1, …)                                          | Apply [`CSwapGate`](qiskit.circuit.library.CSwapGate#qiskit.circuit.library.CSwapGate "qiskit.circuit.library.CSwapGate").          |
| [`from_qasm_file`](#qiskit.circuit.library.TwoLocal.from_qasm_file "qiskit.circuit.library.TwoLocal.from_qasm_file")(path)                                                  | Take in a QASM file and generate a QuantumCircuit object.                                                                           |
| [`from_qasm_str`](#qiskit.circuit.library.TwoLocal.from_qasm_str "qiskit.circuit.library.TwoLocal.from_qasm_str")(qasm\_str)                                                | Take in a QASM string and generate a QuantumCircuit object.                                                                         |
| [`get_entangler_map`](#qiskit.circuit.library.TwoLocal.get_entangler_map "qiskit.circuit.library.TwoLocal.get_entangler_map")(rep\_num, block\_num, …)                      | Overloading to handle the special case of 1 qubit where the entanglement are ignored.                                               |
| [`get_unentangled_qubits`](#qiskit.circuit.library.TwoLocal.get_unentangled_qubits "qiskit.circuit.library.TwoLocal.get_unentangled_qubits")()                              | Get the indices of unentangled qubits in a set.                                                                                     |
| [`h`](#qiskit.circuit.library.TwoLocal.h "qiskit.circuit.library.TwoLocal.h")(qubit)                                                                                        | Apply [`HGate`](qiskit.circuit.library.HGate#qiskit.circuit.library.HGate "qiskit.circuit.library.HGate").                          |
| [`hamiltonian`](#qiskit.circuit.library.TwoLocal.hamiltonian "qiskit.circuit.library.TwoLocal.hamiltonian")(operator, time, qubits\[, label])                               | Apply hamiltonian evolution to to qubits.                                                                                           |
| [`has_register`](#qiskit.circuit.library.TwoLocal.has_register "qiskit.circuit.library.TwoLocal.has_register")(register)                                                    | Test if this circuit has the register r.                                                                                            |
| [`i`](#qiskit.circuit.library.TwoLocal.i "qiskit.circuit.library.TwoLocal.i")(qubit)                                                                                        | Apply [`IGate`](qiskit.circuit.library.IGate#qiskit.circuit.library.IGate "qiskit.circuit.library.IGate").                          |
| [`id`](#qiskit.circuit.library.TwoLocal.id "qiskit.circuit.library.TwoLocal.id")(qubit)                                                                                     | Apply [`IGate`](qiskit.circuit.library.IGate#qiskit.circuit.library.IGate "qiskit.circuit.library.IGate").                          |
| [`initialize`](#qiskit.circuit.library.TwoLocal.initialize "qiskit.circuit.library.TwoLocal.initialize")(params, qubits)                                                    | Apply initialize to circuit.                                                                                                        |
| [`inverse`](#qiskit.circuit.library.TwoLocal.inverse "qiskit.circuit.library.TwoLocal.inverse")()                                                                           | Invert (take adjoint of) this circuit.                                                                                              |
| [`iso`](#qiskit.circuit.library.TwoLocal.iso "qiskit.circuit.library.TwoLocal.iso")(isometry, q\_input, q\_ancillas\_for\_output)                                           | Attach an arbitrary isometry from m to n qubits to a circuit.                                                                       |
| [`isometry`](#qiskit.circuit.library.TwoLocal.isometry "qiskit.circuit.library.TwoLocal.isometry")(isometry, q\_input, …\[, …])                                             | Attach an arbitrary isometry from m to n qubits to a circuit.                                                                       |
| [`iswap`](#qiskit.circuit.library.TwoLocal.iswap "qiskit.circuit.library.TwoLocal.iswap")(qubit1, qubit2)                                                                   | Apply [`iSwapGate`](qiskit.circuit.library.iSwapGate#qiskit.circuit.library.iSwapGate "qiskit.circuit.library.iSwapGate").          |
| [`mcmt`](#qiskit.circuit.library.TwoLocal.mcmt "qiskit.circuit.library.TwoLocal.mcmt")(gate, control\_qubits, target\_qubits\[, …])                                         | Apply a multi-control, multi-target using a generic gate.                                                                           |
| [`mcp`](#qiskit.circuit.library.TwoLocal.mcp "qiskit.circuit.library.TwoLocal.mcp")(lam, control\_qubits, target\_qubit)                                                    | Apply [`MCPhaseGate`](qiskit.circuit.library.MCPhaseGate#qiskit.circuit.library.MCPhaseGate "qiskit.circuit.library.MCPhaseGate").  |
| [`mcrx`](#qiskit.circuit.library.TwoLocal.mcrx "qiskit.circuit.library.TwoLocal.mcrx")(theta, q\_controls, q\_target\[, …])                                                 | Apply Multiple-Controlled X rotation gate                                                                                           |
| [`mcry`](#qiskit.circuit.library.TwoLocal.mcry "qiskit.circuit.library.TwoLocal.mcry")(theta, q\_controls, q\_target, q\_ancillae)                                          | Apply Multiple-Controlled Y rotation gate                                                                                           |
| [`mcrz`](#qiskit.circuit.library.TwoLocal.mcrz "qiskit.circuit.library.TwoLocal.mcrz")(lam, q\_controls, q\_target\[, …])                                                   | Apply Multiple-Controlled Z rotation gate                                                                                           |
| [`mct`](#qiskit.circuit.library.TwoLocal.mct "qiskit.circuit.library.TwoLocal.mct")(control\_qubits, target\_qubit\[, …])                                                   | Apply [`MCXGate`](qiskit.circuit.library.MCXGate#qiskit.circuit.library.MCXGate "qiskit.circuit.library.MCXGate").                  |
| [`mcu1`](#qiskit.circuit.library.TwoLocal.mcu1 "qiskit.circuit.library.TwoLocal.mcu1")(lam, control\_qubits, target\_qubit)                                                 | Apply `MCU1Gate`.                                                                                                                   |
| [`mcx`](#qiskit.circuit.library.TwoLocal.mcx "qiskit.circuit.library.TwoLocal.mcx")(control\_qubits, target\_qubit\[, …])                                                   | Apply [`MCXGate`](qiskit.circuit.library.MCXGate#qiskit.circuit.library.MCXGate "qiskit.circuit.library.MCXGate").                  |
| [`measure`](#qiskit.circuit.library.TwoLocal.measure "qiskit.circuit.library.TwoLocal.measure")(qubit, cbit)                                                                | Measure quantum bit into classical bit (tuples).                                                                                    |
| [`measure_active`](#qiskit.circuit.library.TwoLocal.measure_active "qiskit.circuit.library.TwoLocal.measure_active")(\[inplace])                                            | Adds measurement to all non-idle qubits.                                                                                            |
| [`measure_all`](#qiskit.circuit.library.TwoLocal.measure_all "qiskit.circuit.library.TwoLocal.measure_all")(\[inplace])                                                     | Adds measurement to all qubits.                                                                                                     |
| [`mirror`](#qiskit.circuit.library.TwoLocal.mirror "qiskit.circuit.library.TwoLocal.mirror")()                                                                              | DEPRECATED: use circuit.reverse\_ops().                                                                                             |
| [`ms`](#qiskit.circuit.library.TwoLocal.ms "qiskit.circuit.library.TwoLocal.ms")(theta, qubits)                                                                             | Apply [`MSGate`](qiskit.circuit.library.MSGate#qiskit.circuit.library.MSGate "qiskit.circuit.library.MSGate").                      |
| [`num_connected_components`](#qiskit.circuit.library.TwoLocal.num_connected_components "qiskit.circuit.library.TwoLocal.num_connected_components")(\[unitary\_only])        | How many non-entangled subcircuits can the circuit be factored to.                                                                  |
| [`num_nonlocal_gates`](#qiskit.circuit.library.TwoLocal.num_nonlocal_gates "qiskit.circuit.library.TwoLocal.num_nonlocal_gates")()                                          | Return number of non-local gates (i.e.                                                                                              |
| [`num_tensor_factors`](#qiskit.circuit.library.TwoLocal.num_tensor_factors "qiskit.circuit.library.TwoLocal.num_tensor_factors")()                                          | Computes the number of tensor factors in the unitary (quantum) part of the circuit only.                                            |
| [`num_unitary_factors`](#qiskit.circuit.library.TwoLocal.num_unitary_factors "qiskit.circuit.library.TwoLocal.num_unitary_factors")()                                       | Computes the number of tensor factors in the unitary (quantum) part of the circuit only.                                            |
| [`p`](#qiskit.circuit.library.TwoLocal.p "qiskit.circuit.library.TwoLocal.p")(theta, qubit)                                                                                 | Apply [`PhaseGate`](qiskit.circuit.library.PhaseGate#qiskit.circuit.library.PhaseGate "qiskit.circuit.library.PhaseGate").          |
| [`power`](#qiskit.circuit.library.TwoLocal.power "qiskit.circuit.library.TwoLocal.power")(power\[, matrix\_power])                                                          | Raise this circuit to the power of `power`.                                                                                         |
| [`print_settings`](#qiskit.circuit.library.TwoLocal.print_settings "qiskit.circuit.library.TwoLocal.print_settings")()                                                      | Returns information about the setting.                                                                                              |
| [`qasm`](#qiskit.circuit.library.TwoLocal.qasm "qiskit.circuit.library.TwoLocal.qasm")(\[formatted, filename])                                                              | Return OpenQASM string.                                                                                                             |
| [`qbit_argument_conversion`](#qiskit.circuit.library.TwoLocal.qbit_argument_conversion "qiskit.circuit.library.TwoLocal.qbit_argument_conversion")(qubit\_representation)   | Converts several qubit representations (such as indexes, range, etc.) into a list of qubits.                                        |
| [`qubit_duration`](#qiskit.circuit.library.TwoLocal.qubit_duration "qiskit.circuit.library.TwoLocal.qubit_duration")(\*qubits)                                              | Return the duration between the start and stop time of the first and last instructions, excluding delays, over the supplied qubits. |
| [`qubit_start_time`](#qiskit.circuit.library.TwoLocal.qubit_start_time "qiskit.circuit.library.TwoLocal.qubit_start_time")(\*qubits)                                        | Return the start time of the first instruction, excluding delays, over the supplied qubits.                                         |
| [`qubit_stop_time`](#qiskit.circuit.library.TwoLocal.qubit_stop_time "qiskit.circuit.library.TwoLocal.qubit_stop_time")(\*qubits)                                           | Return the stop time of the last instruction, excluding delays, over the supplied qubits.                                           |
| [`r`](#qiskit.circuit.library.TwoLocal.r "qiskit.circuit.library.TwoLocal.r")(theta, phi, qubit)                                                                            | Apply `RGate`.                                                                                                                      |
| [`rcccx`](#qiskit.circuit.library.TwoLocal.rcccx "qiskit.circuit.library.TwoLocal.rcccx")(control\_qubit1, control\_qubit2, …)                                              | Apply [`RC3XGate`](qiskit.circuit.library.RC3XGate#qiskit.circuit.library.RC3XGate "qiskit.circuit.library.RC3XGate").              |
| [`rccx`](#qiskit.circuit.library.TwoLocal.rccx "qiskit.circuit.library.TwoLocal.rccx")(control\_qubit1, control\_qubit2, …)                                                 | Apply [`RCCXGate`](qiskit.circuit.library.RCCXGate#qiskit.circuit.library.RCCXGate "qiskit.circuit.library.RCCXGate").              |
| [`remove_final_measurements`](#qiskit.circuit.library.TwoLocal.remove_final_measurements "qiskit.circuit.library.TwoLocal.remove_final_measurements")(\[inplace])           | Removes final measurement on all qubits if they are present.                                                                        |
| [`repeat`](#qiskit.circuit.library.TwoLocal.repeat "qiskit.circuit.library.TwoLocal.repeat")(reps)                                                                          | Repeat this circuit `reps` times.                                                                                                   |
| [`reset`](#qiskit.circuit.library.TwoLocal.reset "qiskit.circuit.library.TwoLocal.reset")(qubit)                                                                            | Reset q.                                                                                                                            |
| [`reverse_bits`](#qiskit.circuit.library.TwoLocal.reverse_bits "qiskit.circuit.library.TwoLocal.reverse_bits")()                                                            | Return a circuit with the opposite order of wires.                                                                                  |
| [`reverse_ops`](#qiskit.circuit.library.TwoLocal.reverse_ops "qiskit.circuit.library.TwoLocal.reverse_ops")()                                                               | Reverse the circuit by reversing the order of instructions.                                                                         |
| [`rx`](#qiskit.circuit.library.TwoLocal.rx "qiskit.circuit.library.TwoLocal.rx")(theta, qubit\[, label])                                                                    | Apply [`RXGate`](qiskit.circuit.library.RXGate#qiskit.circuit.library.RXGate "qiskit.circuit.library.RXGate").                      |
| [`rxx`](#qiskit.circuit.library.TwoLocal.rxx "qiskit.circuit.library.TwoLocal.rxx")(theta, qubit1, qubit2)                                                                  | Apply [`RXXGate`](qiskit.circuit.library.RXXGate#qiskit.circuit.library.RXXGate "qiskit.circuit.library.RXXGate").                  |
| [`ry`](#qiskit.circuit.library.TwoLocal.ry "qiskit.circuit.library.TwoLocal.ry")(theta, qubit\[, label])                                                                    | Apply [`RYGate`](qiskit.circuit.library.RYGate#qiskit.circuit.library.RYGate "qiskit.circuit.library.RYGate").                      |
| [`ryy`](#qiskit.circuit.library.TwoLocal.ryy "qiskit.circuit.library.TwoLocal.ryy")(theta, qubit1, qubit2)                                                                  | Apply [`RYYGate`](qiskit.circuit.library.RYYGate#qiskit.circuit.library.RYYGate "qiskit.circuit.library.RYYGate").                  |
| [`rz`](#qiskit.circuit.library.TwoLocal.rz "qiskit.circuit.library.TwoLocal.rz")(phi, qubit)                                                                                | Apply [`RZGate`](qiskit.circuit.library.RZGate#qiskit.circuit.library.RZGate "qiskit.circuit.library.RZGate").                      |
| [`rzx`](#qiskit.circuit.library.TwoLocal.rzx "qiskit.circuit.library.TwoLocal.rzx")(theta, qubit1, qubit2)                                                                  | Apply [`RZXGate`](qiskit.circuit.library.RZXGate#qiskit.circuit.library.RZXGate "qiskit.circuit.library.RZXGate").                  |
| [`rzz`](#qiskit.circuit.library.TwoLocal.rzz "qiskit.circuit.library.TwoLocal.rzz")(theta, qubit1, qubit2)                                                                  | Apply [`RZZGate`](qiskit.circuit.library.RZZGate#qiskit.circuit.library.RZZGate "qiskit.circuit.library.RZZGate").                  |
| [`s`](#qiskit.circuit.library.TwoLocal.s "qiskit.circuit.library.TwoLocal.s")(qubit)                                                                                        | Apply [`SGate`](qiskit.circuit.library.SGate#qiskit.circuit.library.SGate "qiskit.circuit.library.SGate").                          |
| [`sdg`](#qiskit.circuit.library.TwoLocal.sdg "qiskit.circuit.library.TwoLocal.sdg")(qubit)                                                                                  | Apply [`SdgGate`](qiskit.circuit.library.SdgGate#qiskit.circuit.library.SdgGate "qiskit.circuit.library.SdgGate").                  |
| [`size`](#qiskit.circuit.library.TwoLocal.size "qiskit.circuit.library.TwoLocal.size")()                                                                                    | Returns total number of gate operations in circuit.                                                                                 |
| [`snapshot`](#qiskit.circuit.library.TwoLocal.snapshot "qiskit.circuit.library.TwoLocal.snapshot")(label\[, snapshot\_type, qubits, params])                                | Take a statevector snapshot of the internal simulator representation.                                                               |
| [`snapshot_density_matrix`](#qiskit.circuit.library.TwoLocal.snapshot_density_matrix "qiskit.circuit.library.TwoLocal.snapshot_density_matrix")(label\[, qubits])           | Take a density matrix snapshot of simulator state.                                                                                  |
| [`snapshot_expectation_value`](#qiskit.circuit.library.TwoLocal.snapshot_expectation_value "qiskit.circuit.library.TwoLocal.snapshot_expectation_value")(label, op, qubits) | Take a snapshot of expectation value \<O> of an Operator.                                                                           |
| [`snapshot_probabilities`](#qiskit.circuit.library.TwoLocal.snapshot_probabilities "qiskit.circuit.library.TwoLocal.snapshot_probabilities")(label, qubits\[, variance])    | Take a probability snapshot of the simulator state.                                                                                 |
| [`snapshot_stabilizer`](#qiskit.circuit.library.TwoLocal.snapshot_stabilizer "qiskit.circuit.library.TwoLocal.snapshot_stabilizer")(label)                                  | Take a stabilizer snapshot of the simulator state.                                                                                  |
| [`snapshot_statevector`](#qiskit.circuit.library.TwoLocal.snapshot_statevector "qiskit.circuit.library.TwoLocal.snapshot_statevector")(label)                               | Take a statevector snapshot of the simulator state.                                                                                 |
| [`squ`](#qiskit.circuit.library.TwoLocal.squ "qiskit.circuit.library.TwoLocal.squ")(unitary\_matrix, qubit\[, mode, …])                                                     | Decompose an arbitrary 2\*2 unitary into three rotation gates.                                                                      |
| [`swap`](#qiskit.circuit.library.TwoLocal.swap "qiskit.circuit.library.TwoLocal.swap")(qubit1, qubit2)                                                                      | Apply [`SwapGate`](qiskit.circuit.library.SwapGate#qiskit.circuit.library.SwapGate "qiskit.circuit.library.SwapGate").              |
| [`sx`](#qiskit.circuit.library.TwoLocal.sx "qiskit.circuit.library.TwoLocal.sx")(qubit)                                                                                     | Apply [`SXGate`](qiskit.circuit.library.SXGate#qiskit.circuit.library.SXGate "qiskit.circuit.library.SXGate").                      |
| [`sxdg`](#qiskit.circuit.library.TwoLocal.sxdg "qiskit.circuit.library.TwoLocal.sxdg")(qubit)                                                                               | Apply [`SXdgGate`](qiskit.circuit.library.SXdgGate#qiskit.circuit.library.SXdgGate "qiskit.circuit.library.SXdgGate").              |
| [`t`](#qiskit.circuit.library.TwoLocal.t "qiskit.circuit.library.TwoLocal.t")(qubit)                                                                                        | Apply [`TGate`](qiskit.circuit.library.TGate#qiskit.circuit.library.TGate "qiskit.circuit.library.TGate").                          |
| [`tdg`](#qiskit.circuit.library.TwoLocal.tdg "qiskit.circuit.library.TwoLocal.tdg")(qubit)                                                                                  | Apply [`TdgGate`](qiskit.circuit.library.TdgGate#qiskit.circuit.library.TdgGate "qiskit.circuit.library.TdgGate").                  |
| [`to_gate`](#qiskit.circuit.library.TwoLocal.to_gate "qiskit.circuit.library.TwoLocal.to_gate")(\[parameter\_map, label])                                                   | Create a Gate out of this circuit.                                                                                                  |
| [`to_instruction`](#qiskit.circuit.library.TwoLocal.to_instruction "qiskit.circuit.library.TwoLocal.to_instruction")(\[parameter\_map])                                     | Create an Instruction out of this circuit.                                                                                          |
| [`toffoli`](#qiskit.circuit.library.TwoLocal.toffoli "qiskit.circuit.library.TwoLocal.toffoli")(control\_qubit1, control\_qubit2, …)                                        | Apply [`CCXGate`](qiskit.circuit.library.CCXGate#qiskit.circuit.library.CCXGate "qiskit.circuit.library.CCXGate").                  |
| [`u`](#qiskit.circuit.library.TwoLocal.u "qiskit.circuit.library.TwoLocal.u")(theta, phi, lam, qubit)                                                                       | Apply [`UGate`](qiskit.circuit.library.UGate#qiskit.circuit.library.UGate "qiskit.circuit.library.UGate").                          |
| [`u1`](#qiskit.circuit.library.TwoLocal.u1 "qiskit.circuit.library.TwoLocal.u1")(theta, qubit)                                                                              | Apply [`U1Gate`](qiskit.circuit.library.U1Gate#qiskit.circuit.library.U1Gate "qiskit.circuit.library.U1Gate").                      |
| [`u2`](#qiskit.circuit.library.TwoLocal.u2 "qiskit.circuit.library.TwoLocal.u2")(phi, lam, qubit)                                                                           | Apply [`U2Gate`](qiskit.circuit.library.U2Gate#qiskit.circuit.library.U2Gate "qiskit.circuit.library.U2Gate").                      |
| [`u3`](#qiskit.circuit.library.TwoLocal.u3 "qiskit.circuit.library.TwoLocal.u3")(theta, phi, lam, qubit)                                                                    | Apply [`U3Gate`](qiskit.circuit.library.U3Gate#qiskit.circuit.library.U3Gate "qiskit.circuit.library.U3Gate").                      |
| [`uc`](#qiskit.circuit.library.TwoLocal.uc "qiskit.circuit.library.TwoLocal.uc")(gate\_list, q\_controls, q\_target\[, …])                                                  | Attach a uniformly controlled gates (also called multiplexed gates) to a circuit.                                                   |
| [`ucrx`](#qiskit.circuit.library.TwoLocal.ucrx "qiskit.circuit.library.TwoLocal.ucrx")(angle\_list, q\_controls, q\_target)                                                 | Attach a uniformly controlled (also called multiplexed) Rx rotation gate to a circuit.                                              |
| [`ucry`](#qiskit.circuit.library.TwoLocal.ucry "qiskit.circuit.library.TwoLocal.ucry")(angle\_list, q\_controls, q\_target)                                                 | Attach a uniformly controlled (also called multiplexed) Ry rotation gate to a circuit.                                              |
| [`ucrz`](#qiskit.circuit.library.TwoLocal.ucrz "qiskit.circuit.library.TwoLocal.ucrz")(angle\_list, q\_controls, q\_target)                                                 | Attach a uniformly controlled (also called multiplexed gates) Rz rotation gate to a circuit.                                        |
| [`unitary`](#qiskit.circuit.library.TwoLocal.unitary "qiskit.circuit.library.TwoLocal.unitary")(obj, qubits\[, label])                                                      | Apply unitary gate to q.                                                                                                            |
| [`width`](#qiskit.circuit.library.TwoLocal.width "qiskit.circuit.library.TwoLocal.width")()                                                                                 | Return number of qubits plus clbits in circuit.                                                                                     |
| [`x`](#qiskit.circuit.library.TwoLocal.x "qiskit.circuit.library.TwoLocal.x")(qubit\[, label])                                                                              | Apply [`XGate`](qiskit.circuit.library.XGate#qiskit.circuit.library.XGate "qiskit.circuit.library.XGate").                          |
| [`y`](#qiskit.circuit.library.TwoLocal.y "qiskit.circuit.library.TwoLocal.y")(qubit)                                                                                        | Apply [`YGate`](qiskit.circuit.library.YGate#qiskit.circuit.library.YGate "qiskit.circuit.library.YGate").                          |
| [`z`](#qiskit.circuit.library.TwoLocal.z "qiskit.circuit.library.TwoLocal.z")(qubit)                                                                                        | Apply [`ZGate`](qiskit.circuit.library.ZGate#qiskit.circuit.library.ZGate "qiskit.circuit.library.ZGate").                          |

## Attributes

|                                                                                                                                                 |                                                                                                                             |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| [`ancillas`](#qiskit.circuit.library.TwoLocal.ancillas "qiskit.circuit.library.TwoLocal.ancillas")                                              | Returns a list of ancilla bits in the order that the registers were added.                                                  |
| [`calibrations`](#qiskit.circuit.library.TwoLocal.calibrations "qiskit.circuit.library.TwoLocal.calibrations")                                  | Return calibration dictionary.                                                                                              |
| [`clbits`](#qiskit.circuit.library.TwoLocal.clbits "qiskit.circuit.library.TwoLocal.clbits")                                                    | Returns a list of classical bits in the order that the registers were added.                                                |
| [`data`](#qiskit.circuit.library.TwoLocal.data "qiskit.circuit.library.TwoLocal.data")                                                          | Return the circuit data (instructions and context).                                                                         |
| [`entanglement`](#qiskit.circuit.library.TwoLocal.entanglement "qiskit.circuit.library.TwoLocal.entanglement")                                  | Get the entanglement strategy.                                                                                              |
| [`entanglement_blocks`](#qiskit.circuit.library.TwoLocal.entanglement_blocks "qiskit.circuit.library.TwoLocal.entanglement_blocks")             | The blocks in the entanglement layers.                                                                                      |
| `extension_lib`                                                                                                                                 |                                                                                                                             |
| [`global_phase`](#qiskit.circuit.library.TwoLocal.global_phase "qiskit.circuit.library.TwoLocal.global_phase")                                  | Return the global phase of the circuit in radians.                                                                          |
| `header`                                                                                                                                        |                                                                                                                             |
| [`initial_state`](#qiskit.circuit.library.TwoLocal.initial_state "qiskit.circuit.library.TwoLocal.initial_state")                               | Return the initial state that is added in front of the n-local circuit.                                                     |
| [`insert_barriers`](#qiskit.circuit.library.TwoLocal.insert_barriers "qiskit.circuit.library.TwoLocal.insert_barriers")                         | If barriers are inserted in between the layers or not.                                                                      |
| `instances`                                                                                                                                     |                                                                                                                             |
| [`num_ancillas`](#qiskit.circuit.library.TwoLocal.num_ancillas "qiskit.circuit.library.TwoLocal.num_ancillas")                                  | Return the number of ancilla qubits.                                                                                        |
| [`num_clbits`](#qiskit.circuit.library.TwoLocal.num_clbits "qiskit.circuit.library.TwoLocal.num_clbits")                                        | Return number of classical bits.                                                                                            |
| [`num_layers`](#qiskit.circuit.library.TwoLocal.num_layers "qiskit.circuit.library.TwoLocal.num_layers")                                        | Return the number of layers in the n-local circuit.                                                                         |
| [`num_parameters`](#qiskit.circuit.library.TwoLocal.num_parameters "qiskit.circuit.library.TwoLocal.num_parameters")                            | Convenience function to get the number of parameter objects in the circuit.                                                 |
| [`num_parameters_settable`](#qiskit.circuit.library.TwoLocal.num_parameters_settable "qiskit.circuit.library.TwoLocal.num_parameters_settable") | The number of total parameters that can be set to distinct values.                                                          |
| [`num_qubits`](#qiskit.circuit.library.TwoLocal.num_qubits "qiskit.circuit.library.TwoLocal.num_qubits")                                        | Returns the number of qubits in this circuit.                                                                               |
| [`ordered_parameters`](#qiskit.circuit.library.TwoLocal.ordered_parameters "qiskit.circuit.library.TwoLocal.ordered_parameters")                | The parameters used in the underlying circuit.                                                                              |
| [`parameter_bounds`](#qiskit.circuit.library.TwoLocal.parameter_bounds "qiskit.circuit.library.TwoLocal.parameter_bounds")                      | The parameter bounds for the unbound parameters in the circuit.                                                             |
| [`parameters`](#qiskit.circuit.library.TwoLocal.parameters "qiskit.circuit.library.TwoLocal.parameters")                                        | Get the [`Parameter`](qiskit.circuit.Parameter#qiskit.circuit.Parameter "qiskit.circuit.Parameter") objects in the circuit. |
| [`preferred_init_points`](#qiskit.circuit.library.TwoLocal.preferred_init_points "qiskit.circuit.library.TwoLocal.preferred_init_points")       | The initial points for the parameters.                                                                                      |
| `prefix`                                                                                                                                        |                                                                                                                             |
| [`qregs`](#qiskit.circuit.library.TwoLocal.qregs "qiskit.circuit.library.TwoLocal.qregs")                                                       | A list of the quantum registers associated with the circuit.                                                                |
| [`qubits`](#qiskit.circuit.library.TwoLocal.qubits "qiskit.circuit.library.TwoLocal.qubits")                                                    | Returns a list of quantum bits in the order that the registers were added.                                                  |
| [`reps`](#qiskit.circuit.library.TwoLocal.reps "qiskit.circuit.library.TwoLocal.reps")                                                          | The number of times rotation and entanglement block are repeated.                                                           |
| [`rotation_blocks`](#qiskit.circuit.library.TwoLocal.rotation_blocks "qiskit.circuit.library.TwoLocal.rotation_blocks")                         | The blocks in the rotation layers.                                                                                          |

<span id="undefined" />

`add_calibration(gate, qubits, schedule, params=None)`

Register a low-level, custom pulse definition for the given gate.

**Parameters**

*   **gate** (*Union\[*[*Gate*](qiskit.circuit.Gate#qiskit.circuit.Gate "qiskit.circuit.Gate")*, str]*) – Gate information.
*   **qubits** (*Union\[int, Tuple\[int]]*) – List of qubits to be measured.
*   **schedule** ([*Schedule*](qiskit.pulse.Schedule#qiskit.pulse.Schedule "qiskit.pulse.Schedule")) – Schedule information.
*   **params** (*Optional\[List\[Union\[float,* [*Parameter*](qiskit.circuit.Parameter#qiskit.circuit.Parameter "qiskit.circuit.Parameter")*]]]*) – A list of parameters.

**Raises**

**Exception** – if the gate is of type string and params is None.

<span id="undefined" />

`add_layer(other, entanglement=None, front=False)`

Append another layer to the NLocal.

**Parameters**

*   **other** (`Union`\[`NLocal`, `Instruction`, `QuantumCircuit`]) – The layer to compose, can be another NLocal, an Instruction or Gate, or a QuantumCircuit.
*   **entanglement** (`Union`\[`List`\[`int`], `str`, `List`\[`List`\[`int`]], `None`]) – The entanglement or qubit indices.
*   **front** (`bool`) – If True, `other` is appended to the front, else to the back.

**Return type**

`NLocal`

**Returns**

self, such that chained composes are possible.

**Raises**

**TypeError** – If other is not compatible, i.e. is no Instruction and does not have a to\_instruction method.

<span id="undefined" />

`add_register(*regs)`

Add registers.

<span id="undefined" />

`property ancillas`

Returns a list of ancilla bits in the order that the registers were added.

<span id="undefined" />

`append(instruction, qargs=None, cargs=None)`

Append one or more instructions to the end of the circuit, modifying the circuit in place. Expands qargs and cargs.

**Parameters**

*   **instruction** ([*qiskit.circuit.Instruction*](qiskit.circuit.Instruction#qiskit.circuit.Instruction "qiskit.circuit.Instruction")) – Instruction instance to append
*   **qargs** (*list(argument)*) – qubits to attach instruction to
*   **cargs** (*list(argument)*) – clbits to attach instruction to

**Returns**

a handle to the instruction that was just added

**Return type**

[qiskit.circuit.Instruction](qiskit.circuit.Instruction#qiskit.circuit.Instruction "qiskit.circuit.Instruction")

**Raises**

*   **CircuitError** – if object passed is a subclass of Instruction
*   **CircuitError** – if object passed is neither subclass nor an instance of Instruction

<span id="undefined" />

`assign_parameters(param_dict, inplace=False)`

Assign parameters to the n-local circuit.

This method also supports passing a list instead of a dictionary. If a list is passed, the list must have the same length as the number of unbound parameters in the circuit. The parameters are assigned in the order of the parameters in [`ordered_parameters()`](#qiskit.circuit.library.TwoLocal.ordered_parameters "qiskit.circuit.library.TwoLocal.ordered_parameters").

**Return type**

`Optional`\[`QuantumCircuit`]

**Returns**

A copy of the NLocal circuit with the specified parameters.

**Raises**

**AttributeError** – If the parameters are given as list and do not match the number of parameters.

<span id="undefined" />

`barrier(*qargs)`

Apply `Barrier`. If qargs is None, applies to all.

<span id="undefined" />

`bind_parameters(value_dict)`

Assign numeric parameters to values yielding a new circuit.

To assign new Parameter objects or bind the values in-place, without yielding a new circuit, use the [`assign_parameters()`](#qiskit.circuit.library.TwoLocal.assign_parameters "qiskit.circuit.library.TwoLocal.assign_parameters") method.

**Parameters**

**value\_dict** (*dict*) – \{parameter: value, …}

**Raises**

*   **CircuitError** – If value\_dict contains parameters not present in the circuit
*   **TypeError** – If value\_dict contains a ParameterExpression in the values.

**Returns**

copy of self with assignment substitution.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`property calibrations`

Return calibration dictionary.

**The custom pulse definition of a given gate is of the form**

\{‘gate\_name’: \{(qubits, params): schedule}}

<span id="undefined" />

`static cast(value, _type)`

Best effort to cast value to type. Otherwise, returns the value.

<span id="undefined" />

`cbit_argument_conversion(clbit_representation)`

Converts several classical bit representations (such as indexes, range, etc.) into a list of classical bits.

**Parameters**

**clbit\_representation** (*Object*) – representation to expand

**Returns**

Where each tuple is a classical bit.

**Return type**

List(tuple)

<span id="undefined" />

`ccx(control_qubit1, control_qubit2, target_qubit)`

Apply [`CCXGate`](qiskit.circuit.library.CCXGate#qiskit.circuit.library.CCXGate "qiskit.circuit.library.CCXGate").

<span id="undefined" />

`ch(control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CHGate`](qiskit.circuit.library.CHGate#qiskit.circuit.library.CHGate "qiskit.circuit.library.CHGate").

<span id="undefined" />

`property clbits`

Returns a list of classical bits in the order that the registers were added.

<span id="undefined" />

`classmethod cls_instances()`

Return the current number of instances of this class, useful for auto naming.

<span id="undefined" />

`classmethod cls_prefix()`

Return the prefix to use for auto naming.

<span id="undefined" />

`cnot(control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CXGate`](qiskit.circuit.library.CXGate#qiskit.circuit.library.CXGate "qiskit.circuit.library.CXGate").

<span id="undefined" />

`combine(rhs)`

Append rhs to self if self contains compatible registers.

Two circuits are compatible if they contain the same registers or if they contain different registers with unique names. The returned circuit will contain all unique registers between both circuits.

Return self + rhs as a new object.

**Parameters**

**rhs** ([*QuantumCircuit*](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")) – The quantum circuit to append to the right hand side.

**Returns**

Returns a new QuantumCircuit object

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the rhs circuit is not compatible

<span id="undefined" />

`compose(other, qubits=None, clbits=None, front=False, inplace=False)`

Compose circuit with `other` circuit or instruction, optionally permuting wires.

`other` can be narrower or of equal width to `self`.

**Parameters**

*   **other** ([*qiskit.circuit.Instruction*](qiskit.circuit.Instruction#qiskit.circuit.Instruction "qiskit.circuit.Instruction")  *or*[*QuantumCircuit*](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit") *or BaseOperator*) – (sub)circuit to compose onto self.
*   **qubits** (*list\[Qubit|int]*) – qubits of self to compose onto.
*   **clbits** (*list\[Clbit|int]*) – clbits of self to compose onto.
*   **front** (*bool*) – If True, front composition will be performed (not implemented yet).
*   **inplace** (*bool*) – If True, modify the object. Otherwise return composed circuit.

**Returns**

the composed circuit (returns None if inplace==True).

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

*   **CircuitError** – if composing on the front.
*   **QiskitError** – if `other` is wider or there are duplicate edge mappings.

## Examples

```python
>>> lhs.compose(rhs, qubits=[3, 2], inplace=True)
```

```python
            ┌───┐                   ┌─────┐                ┌───┐
lqr_1_0: ───┤ H ├───    rqr_0: ──■──┤ Tdg ├    lqr_1_0: ───┤ H ├───────────────
            ├───┤              ┌─┴─┐└─────┘                ├───┤
lqr_1_1: ───┤ X ├───    rqr_1: ┤ X ├───────    lqr_1_1: ───┤ X ├───────────────
         ┌──┴───┴──┐           └───┘                    ┌──┴───┴──┐┌───┐
lqr_1_2: ┤ U1(0.1) ├  +                     =  lqr_1_2: ┤ U1(0.1) ├┤ X ├───────
         └─────────┘                                    └─────────┘└─┬─┘┌─────┐
lqr_2_0: ─────■─────                           lqr_2_0: ─────■───────■──┤ Tdg ├
            ┌─┴─┐                                          ┌─┴─┐        └─────┘
lqr_2_1: ───┤ X ├───                           lqr_2_1: ───┤ X ├───────────────
            └───┘                                          └───┘
lcr_0: 0 ═══════════                           lcr_0: 0 ═══════════════════════

lcr_1: 0 ═══════════                           lcr_1: 0 ═══════════════════════
```

<span id="undefined" />

`control(num_ctrl_qubits=1, label=None, ctrl_state=None)`

Control this circuit on `num_ctrl_qubits` qubits.

**Parameters**

*   **num\_ctrl\_qubits** (*int*) – The number of control qubits.
*   **label** (*str*) – An optional label to give the controlled operation for visualization.
*   **ctrl\_state** (*str or int*) – The control state in decimal or as a bitstring (e.g. ‘111’). If None, use `2**num_ctrl_qubits - 1`.

**Returns**

The controlled version of this circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**CircuitError** – If the circuit contains a non-unitary operation and cannot be controlled.

<span id="undefined" />

`copy(name=None)`

Copy the circuit.

**Parameters**

**name** (*str*) – name to be given to the copied circuit. If None, then the name stays the same

**Returns**

a deepcopy of the current circuit, with the specified name

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`count_ops()`

Count each operation kind in the circuit.

**Returns**

a breakdown of how many operations of each kind, sorted by amount.

**Return type**

OrderedDict

<span id="undefined" />

`cp(theta, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CPhaseGate`](qiskit.circuit.library.CPhaseGate#qiskit.circuit.library.CPhaseGate "qiskit.circuit.library.CPhaseGate").

<span id="undefined" />

`crx(theta, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CRXGate`](qiskit.circuit.library.CRXGate#qiskit.circuit.library.CRXGate "qiskit.circuit.library.CRXGate").

<span id="undefined" />

`cry(theta, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CRYGate`](qiskit.circuit.library.CRYGate#qiskit.circuit.library.CRYGate "qiskit.circuit.library.CRYGate").

<span id="undefined" />

`crz(theta, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CRZGate`](qiskit.circuit.library.CRZGate#qiskit.circuit.library.CRZGate "qiskit.circuit.library.CRZGate").

<span id="undefined" />

`cswap(control_qubit, target_qubit1, target_qubit2, label=None, ctrl_state=None)`

Apply [`CSwapGate`](qiskit.circuit.library.CSwapGate#qiskit.circuit.library.CSwapGate "qiskit.circuit.library.CSwapGate").

<span id="undefined" />

`csx(control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CSXGate`](qiskit.circuit.library.CSXGate#qiskit.circuit.library.CSXGate "qiskit.circuit.library.CSXGate").

<span id="undefined" />

`cu(theta, phi, lam, gamma, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CUGate`](qiskit.circuit.library.CUGate#qiskit.circuit.library.CUGate "qiskit.circuit.library.CUGate").

<span id="undefined" />

`cu1(theta, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CU1Gate`](qiskit.circuit.library.CU1Gate#qiskit.circuit.library.CU1Gate "qiskit.circuit.library.CU1Gate").

<span id="undefined" />

`cu3(theta, phi, lam, control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CU3Gate`](qiskit.circuit.library.CU3Gate#qiskit.circuit.library.CU3Gate "qiskit.circuit.library.CU3Gate").

<span id="undefined" />

`cx(control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CXGate`](qiskit.circuit.library.CXGate#qiskit.circuit.library.CXGate "qiskit.circuit.library.CXGate").

<span id="undefined" />

`cy(control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CYGate`](qiskit.circuit.library.CYGate#qiskit.circuit.library.CYGate "qiskit.circuit.library.CYGate").

<span id="undefined" />

`cz(control_qubit, target_qubit, label=None, ctrl_state=None)`

Apply [`CZGate`](qiskit.circuit.library.CZGate#qiskit.circuit.library.CZGate "qiskit.circuit.library.CZGate").

<span id="undefined" />

`property data`

Return the circuit data (instructions and context).

**Returns**

a list-like object containing the tuples for the circuit’s data.

Each tuple is in the format `(instruction, qargs, cargs)`, where instruction is an Instruction (or subclass) object, qargs is a list of Qubit objects, and cargs is a list of Clbit objects.

**Return type**

QuantumCircuitData

<span id="undefined" />

`dcx(qubit1, qubit2)`

Apply [`DCXGate`](qiskit.circuit.library.DCXGate#qiskit.circuit.library.DCXGate "qiskit.circuit.library.DCXGate").

<span id="undefined" />

`decompose()`

Call a decomposition pass on this circuit, to decompose one level (shallow decompose).

**Returns**

a circuit one level decomposed

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`delay(duration, qarg=None, unit='dt')`

Apply [`Delay`](qiskit.circuit.Delay#qiskit.circuit.Delay "qiskit.circuit.Delay"). If qarg is None, applies to all qubits. When applying to multiple qubits, delays with the same duration will be created.

**Parameters**

*   **duration** (*int or float*) – duration of the delay.
*   **qarg** (*Object*) – qubit argument to apply this delay.
*   **unit** (*str*) – unit of the duration. Supported units: ‘s’, ‘ms’, ‘us’, ‘ns’, ‘ps’, ‘dt’. Default is `dt`, i.e. integer time unit depending on the target backend.

**Returns**

the attached delay instruction.

**Return type**

qiskit.Instruction

**Raises**

**CircuitError** – if arguments have bad format.

<span id="undefined" />

`depth()`

Return circuit depth (i.e., length of critical path). This does not include compiler or simulator directives such as ‘barrier’ or ‘snapshot’.

**Returns**

Depth of circuit.

**Return type**

int

## Notes

The circuit depth and the DAG depth need not be the same.

<span id="undefined" />

`diag_gate(diag, qubit)`

Deprecated version of QuantumCircuit.diagonal.

<span id="undefined" />

`diagonal(diag, qubit)`

Attach a diagonal gate to a circuit.

The decomposition is based on Theorem 7 given in “Synthesis of Quantum Logic Circuits” by Shende et al. ([https://arxiv.org/pdf/quant-ph/0406176.pdf](https://arxiv.org/pdf/quant-ph/0406176.pdf)).

**Parameters**

*   **diag** (*list*) – list of the 2^k diagonal entries (for a diagonal gate on k qubits). Must contain at least two entries
*   **qubit** (*QuantumRegister|list*) – list of k qubits the diagonal is acting on (the order of the qubits specifies the computational basis in which the diagonal gate is provided: the first element in diag acts on the state where all the qubits in q are in the state 0, the second entry acts on the state where all the qubits q\[1],…,q\[k-1] are in the state zero and q\[0] is in the state 1, and so on)

**Returns**

the diagonal gate which was attached to the circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the list of the diagonal entries or the qubit list is in bad format; if the number of diagonal entries is not 2^k, where k denotes the number of qubits

<span id="undefined" />

`draw(output=None, scale=None, filename=None, style=None, interactive=False, plot_barriers=True, reverse_bits=False, justify=None, vertical_compression='medium', idle_wires=True, with_layout=True, fold=None, ax=None, initial_state=False, cregbundle=True)`

Draw the quantum circuit.

**text**: ASCII art TextDrawing that can be printed in the console.

**latex**: high-quality images compiled via LaTeX.

**latex\_source**: raw uncompiled LaTeX output.

**matplotlib**: images with color rendered purely in Python.

**Parameters**

*   **output** (*str*) – Select the output method to use for drawing the circuit. Valid choices are `text`, `latex`, `latex_source`, or `mpl`. By default the ‘text’ drawer is used unless a user config file has an alternative backend set as the default. If the output kwarg is set, that backend will always be used over the default in a user config file.
*   **scale** (*float*) – scale of image to draw (shrink if \< 1)
*   **filename** (*str*) – file path to save image to
*   **style** (*dict or str*) – dictionary of style or file name of style file. This option is only used by the `mpl` output type. If a str is passed in that is the path to a json file which contains a dictionary of style, then that will be opened, parsed, and used as the input dict. See: [Style Dict Doc](qiskit.circuit.library.ZZFeatureMap#style-dict-circ-doc) for more information on the contents.
*   **interactive** (*bool*) – when set true show the circuit in a new window (for mpl this depends on the matplotlib backend being used supporting this). Note when used with either the text or the latex\_source output type this has no effect and will be silently ignored.
*   **reverse\_bits** (*bool*) – When set to True, reverse the bit order inside registers for the output visualization.
*   **plot\_barriers** (*bool*) – Enable/disable drawing barriers in the output circuit. Defaults to True.
*   **justify** (*string*) – Options are `left`, `right` or `none`. If anything else is supplied it defaults to left justified. It refers to where gates should be placed in the output circuit if there is an option. `none` results in each gate being placed in its own column.
*   **vertical\_compression** (*string*) – `high`, `medium` or `low`. It merges the lines generated by the `text` output so the drawing will take less vertical room. Default is `medium`. Only used by the `text` output, will be silently ignored otherwise.
*   **idle\_wires** (*bool*) – Include idle wires (wires with no circuit elements) in output visualization. Default is True.
*   **with\_layout** (*bool*) – Include layout information, with labels on the physical layout. Default is True.
*   **fold** (*int*) – Sets pagination. It can be disabled using -1. In text, sets the length of the lines. This is useful when the drawing does not fit in the console. If None (default), it will try to guess the console width using `shutil. get_terminal_size()`. However, if running in jupyter, the default line length is set to 80 characters. In `mpl` is the number of (visual) layers before folding. Default is 25.
*   **ax** (*matplotlib.axes.Axes*) – An optional Axes object to be used for the visualization output. If none is specified, a new matplotlib Figure will be created and used. Additionally, if specified, there will be no returned Figure since it is redundant. This is only used when the `output` kwarg is set to use the `mpl` backend. It will be silently ignored with all other outputs.
*   **initial\_state** (*bool*) – Optional. Adds `|0>` in the beginning of the wire. Only used by the `text`, `latex` and `latex_source` outputs. Default: `False`.
*   **cregbundle** (*bool*) – Optional. If set True bundle classical registers. Not used by the `matplotlib` output. Default: `True`.

**Returns**

`PIL.Image` or `matplotlib.figure` or `str` or `TextDrawing`:

*   **PIL.Image (output=’latex’)**

    an in-memory representation of the image of the circuit diagram.

*   **matplotlib.figure.Figure (output=’mpl’)**

    a matplotlib figure object for the circuit diagram.

*   **str (output=’latex\_source’)**

    The LaTeX source code for visualizing the circuit diagram.

*   **TextDrawing (output=’text’)**

    A drawing that can be printed as ASCII art.

**Raises**

*   [**VisualizationError**](qiskit.visualization.VisualizationError#qiskit.visualization.VisualizationError "qiskit.visualization.VisualizationError") – when an invalid output method is selected
*   **ImportError** – when the output methods require non-installed libraries

**Style Dict Details**

The style dict kwarg contains numerous options that define the style of the output circuit visualization. The style dict is only used by the `mpl` output. The options available in the style dict are defined below:

**Parameters**

*   **name** (*str*) – The name of the style. The name can be set to ‘iqx’, ‘bw’, or ‘default’. This overrides the setting in the ‘\~/.qiskit/settings.conf’ file.

*   **textcolor** (*str*) – The color code to use for text. Defaults to ‘#000000’

*   **subtextcolor** (*str*) – The color code to use for subtext. Defaults to ‘#000000’

*   **linecolor** (*str*) – The color code to use for lines. Defaults to ‘#000000’

*   **creglinecolor** (*str*) – The color code to use for classical register lines. Defaults to ‘#778899’

*   **gatetextcolor** (*str*) – The color code to use for gate text. Defaults to ‘#000000’

*   **gatefacecolor** (*str*) – The color code to use for gates. Defaults to ‘#ffffff’

*   **barrierfacecolor** (*str*) – The color code to use for barriers. Defaults to ‘#bdbdbd’

*   **backgroundcolor** (*str*) – The color code to use for the background. Defaults to ‘#ffffff’

*   **fontsize** (*int*) – The font size to use for text. Defaults to 13.

*   **subfontsize** (*int*) – The font size to use for subtext. Defaults to 8.

*   **displaytext** (*dict*) –

    A dictionary of the text to use for each element type in the output visualization. The default values are:

    ```python
    {
        'id': 'id',
        'u0': 'U_0',
        'u1': 'U_1',
        'u2': 'U_2',
        'u3': 'U_3',
        'x': 'X',
        'y': 'Y',
        'z': 'Z',
        'h': 'H',
        's': 'S',
        'sdg': 'S^\dagger',
        't': 'T',
        'tdg': 'T^\dagger',
        'rx': 'R_x',
        'ry': 'R_y',
        'rz': 'R_z',
        'reset': '\left|0\right\rangle'
    }
    ```

    You must specify all the necessary values if using this. There is no provision for passing an incomplete dict in.

*   **displaycolor** (*dict*) –

    The color codes to use for each circuit element in the form (gate\_color, text\_color). The default values are:

    ```python
    {
        'u1': ('#FA74A6', '#000000'),
        'u2': ('#FA74A6', '#000000'),
        'u3': ('#FA74A6', '#000000'),
        'id': ('#05BAB6', '#000000'),
        'x': ('#05BAB6', '#000000'),
        'y': ('#05BAB6', '#000000'),
        'z': ('#05BAB6', '#000000'),
        'h': ('#6FA4FF', '#000000'),
        'cx': ('#6FA4FF', '#000000'),
        'cy': ('#6FA4FF', '#000000'),
        'cz': ('#6FA4FF', '#000000'),
        'swap': ('#6FA4FF', '#000000'),
        's': ('#6FA4FF', '#000000'),
        'sdg': ('#6FA4FF', '#000000'),
        'dcx': ('#6FA4FF', '#000000'),
        'iswap': ('#6FA4FF', '#000000'),
        't': ('#BB8BFF', '#000000'),
        'tdg': ('#BB8BFF', '#000000'),
        'r': ('#BB8BFF', '#000000'),
        'rx': ('#BB8BFF', '#000000'),
        'ry': ('#BB8BFF', '#000000'),
        'rz': ('#BB8BFF', '#000000'),
        'rxx': ('#BB8BFF', '#000000'),
        'ryy': ('#BB8BFF', '#000000'),
        'rzx': ('#BB8BFF', '#000000'),
        'reset': ('#000000', #FFFFFF'),
        'target': ('#FFFFFF, '#FFFFFF'),
        'measure': ('#000000', '#FFFFFF'),
        'ccx': ('#BB8BFF', '#000000'),
        'cdcx': ('#BB8BFF', '#000000'),
        'ccdcx': ('#BB8BFF', '#000000'),
        'cswap': ('#BB8BFF', '#000000'),
        'ccswap': ('#BB8BFF', '#000000'),
        'mcx': ('#BB8BFF', '#000000'),
        'mcx_gray': ('#BB8BFF', '#000000),
        'u': ('#BB8BFF', '#000000'),
        'p': ('#BB8BFF', '#000000'),
        'sx': ('#BB8BFF', '#000000'),
        'sxdg': ('#BB8BFF', '#000000')
    }
    ```

    Colors can also be entered without the text color, such as ‘u1’: ‘#FA74A6’, in which case the text color will always be ‘gatetextcolor’. The ‘displaycolor’ dict can contain any number of elements from one to the entire dict above.

*   **latexdrawerstyle** (*bool*) – When set to True, enable LaTeX mode, which will draw gates like the latex output modes.

*   **usepiformat** (*bool*) – When set to True, use radians for output.

*   **fold** (*int*) – The number of circuit elements to fold the circuit at. Defaults to 20.

*   **cregbundle** (*bool*) – If set True, bundle classical registers

*   **showindex** (*bool*) – If set True, draw an index.

*   **compress** (*bool*) – If set True, draw a compressed circuit.

*   **figwidth** (*int*) – The maximum width (in inches) for the output figure.

*   **dpi** (*int*) – The DPI to use for the output image. Defaults to 150.

*   **margin** (*list*) – A list of margin values to adjust spacing around output image. Takes a list of 4 ints: \[x left, x right, y bottom, y top].

*   **creglinestyle** (*str*) – The style of line to use for classical registers. Choices are ‘solid’, ‘doublet’, or any valid matplotlib linestyle kwarg value. Defaults to doublet

<span id="undefined" />

`property entanglement`

Get the entanglement strategy.

**Return type**

`Union`\[`str`, `List`\[`str`], `List`\[`List`\[`str`]], `List`\[`int`], `List`\[`List`\[`int`]], `List`\[`List`\[`List`\[`int`]]], `List`\[`List`\[`List`\[`List`\[`int`]]]], `Callable`\[\[`int`], `str`], `Callable`\[\[`int`], `List`\[`List`\[`int`]]]]

**Returns**

The entanglement strategy, see [`get_entangler_map()`](#qiskit.circuit.library.TwoLocal.get_entangler_map "qiskit.circuit.library.TwoLocal.get_entangler_map") for more detail on how the format is interpreted.

<span id="undefined" />

`property entanglement_blocks`

The blocks in the entanglement layers.

**Return type**

`List`\[`Instruction`]

**Returns**

The blocks in the entanglement layers.

<span id="undefined" />

`extend(rhs)`

Append QuantumCircuit to the right hand side if it contains compatible registers.

Two circuits are compatible if they contain the same registers or if they contain different registers with unique names. The returned circuit will contain all unique registers between both circuits.

Modify and return self.

**Parameters**

**rhs** ([*QuantumCircuit*](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")) – The quantum circuit to append to the right hand side.

**Returns**

Returns this QuantumCircuit object (which has been modified)

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the rhs circuit is not compatible

<span id="undefined" />

`fredkin(control_qubit, target_qubit1, target_qubit2)`

Apply [`CSwapGate`](qiskit.circuit.library.CSwapGate#qiskit.circuit.library.CSwapGate "qiskit.circuit.library.CSwapGate").

<span id="undefined" />

`static from_qasm_file(path)`

Take in a QASM file and generate a QuantumCircuit object.

**Parameters**

**path** (*str*) – Path to the file for a QASM program

**Returns**

The QuantumCircuit object for the input QASM

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`static from_qasm_str(qasm_str)`

Take in a QASM string and generate a QuantumCircuit object.

**Parameters**

**qasm\_str** (*str*) – A QASM program string

**Returns**

The QuantumCircuit object for the input QASM

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`get_entangler_map(rep_num, block_num, num_block_qubits)`

Overloading to handle the special case of 1 qubit where the entanglement are ignored.

**Return type**

`List`\[`List`\[`int`]]

<span id="undefined" />

`get_unentangled_qubits()`

Get the indices of unentangled qubits in a set.

**Return type**

`Set`\[`int`]

**Returns**

The unentangled qubits.

<span id="undefined" />

`property global_phase`

Return the global phase of the circuit in radians.

<span id="undefined" />

`h(qubit)`

Apply [`HGate`](qiskit.circuit.library.HGate#qiskit.circuit.library.HGate "qiskit.circuit.library.HGate").

<span id="undefined" />

`hamiltonian(operator, time, qubits, label=None)`

Apply hamiltonian evolution to to qubits.

<span id="undefined" />

`has_register(register)`

Test if this circuit has the register r.

**Parameters**

**register** (*Register*) – a quantum or classical register.

**Returns**

True if the register is contained in this circuit.

**Return type**

bool

<span id="undefined" />

`i(qubit)`

Apply [`IGate`](qiskit.circuit.library.IGate#qiskit.circuit.library.IGate "qiskit.circuit.library.IGate").

<span id="undefined" />

`id(qubit)`

Apply [`IGate`](qiskit.circuit.library.IGate#qiskit.circuit.library.IGate "qiskit.circuit.library.IGate").

<span id="undefined" />

`property initial_state`

Return the initial state that is added in front of the n-local circuit.

**Return type**

`Any`

**Returns**

The initial state.

<span id="undefined" />

`initialize(params, qubits)`

Apply initialize to circuit.

<span id="undefined" />

`property insert_barriers`

If barriers are inserted in between the layers or not.

**Return type**

`bool`

**Returns**

True, if barriers are inserted in between the layers, False if not.

<span id="undefined" />

`inverse()`

Invert (take adjoint of) this circuit.

This is done by recursively inverting all gates.

**Returns**

the inverted circuit

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**CircuitError** – if the circuit cannot be inverted.

## Examples

**input:**

┌───┐

**q\_0: ┤ H ├─────■──────**

└───┘┌────┴─────┐

**q\_1: ─────┤ RX(1.57) ├**

└──────────┘

**output:**

┌───┐

**q\_0: ──────■──────┤ H ├**

┌─────┴─────┐└───┘

**q\_1: ┤ RX(-1.57) ├─────**

└───────────┘

<span id="undefined" />

`iso(isometry, q_input, q_ancillas_for_output, q_ancillas_zero=None, q_ancillas_dirty=None)`

Attach an arbitrary isometry from m to n qubits to a circuit. In particular, this allows to attach arbitrary unitaries on n qubits (m=n) or to prepare any state on n qubits (m=0). The decomposition used here was introduced by Iten et al. in [https://arxiv.org/abs/1501.06911](https://arxiv.org/abs/1501.06911).

**Parameters**

*   **isometry** (*ndarray*) – an isometry from m to n qubits, i.e., a (complex) ndarray of dimension 2^n×2^m with orthonormal columns (given in the computational basis specified by the order of the ancillas and the input qubits, where the ancillas are considered to be more significant than the input qubits.).
*   **q\_input** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of m qubits where the input to the isometry is fed in (empty list for state preparation).
*   **q\_ancillas\_for\_output** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of n-m ancilla qubits that are used for the output of the isometry and which are assumed to start in the zero state. The qubits are listed with increasing significance.
*   **q\_ancillas\_zero** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of ancilla qubits which are assumed to start in the zero state. Default is q\_ancillas\_zero = None.
*   **q\_ancillas\_dirty** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of ancilla qubits which can start in an arbitrary state. Default is q\_ancillas\_dirty = None.

**Returns**

the isometry is attached to the quantum circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the array is not an isometry of the correct size corresponding to the provided number of qubits.

<span id="undefined" />

`isometry(isometry, q_input, q_ancillas_for_output, q_ancillas_zero=None, q_ancillas_dirty=None)`

Attach an arbitrary isometry from m to n qubits to a circuit. In particular, this allows to attach arbitrary unitaries on n qubits (m=n) or to prepare any state on n qubits (m=0). The decomposition used here was introduced by Iten et al. in [https://arxiv.org/abs/1501.06911](https://arxiv.org/abs/1501.06911).

**Parameters**

*   **isometry** (*ndarray*) – an isometry from m to n qubits, i.e., a (complex) ndarray of dimension 2^n×2^m with orthonormal columns (given in the computational basis specified by the order of the ancillas and the input qubits, where the ancillas are considered to be more significant than the input qubits.).
*   **q\_input** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of m qubits where the input to the isometry is fed in (empty list for state preparation).
*   **q\_ancillas\_for\_output** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of n-m ancilla qubits that are used for the output of the isometry and which are assumed to start in the zero state. The qubits are listed with increasing significance.
*   **q\_ancillas\_zero** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of ancilla qubits which are assumed to start in the zero state. Default is q\_ancillas\_zero = None.
*   **q\_ancillas\_dirty** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of ancilla qubits which can start in an arbitrary state. Default is q\_ancillas\_dirty = None.

**Returns**

the isometry is attached to the quantum circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the array is not an isometry of the correct size corresponding to the provided number of qubits.

<span id="undefined" />

`iswap(qubit1, qubit2)`

Apply [`iSwapGate`](qiskit.circuit.library.iSwapGate#qiskit.circuit.library.iSwapGate "qiskit.circuit.library.iSwapGate").

<span id="undefined" />

`mcmt(gate, control_qubits, target_qubits, ancilla_qubits=None, mode='noancilla', *, single_control_gate_fun=None, q_controls=None, q_ancillae=None, q_targets=None)`

Apply a multi-control, multi-target using a generic gate.

This can also be used to implement a generic multi-control gate, as the target could also be of length 1.

<span id="undefined" />

`mcp(lam, control_qubits, target_qubit)`

Apply [`MCPhaseGate`](qiskit.circuit.library.MCPhaseGate#qiskit.circuit.library.MCPhaseGate "qiskit.circuit.library.MCPhaseGate").

<span id="undefined" />

`mcrx(theta, q_controls, q_target, use_basis_gates=False)`

Apply Multiple-Controlled X rotation gate

**Parameters**

*   **self** ([*QuantumCircuit*](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")) – The QuantumCircuit object to apply the mcrx gate on.
*   **theta** (*float*) – angle theta
*   **q\_controls** (*list(*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*)*) – The list of control qubits
*   **q\_target** ([*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")) – The target qubit
*   **use\_basis\_gates** (*bool*) – use p, u, cx

**Raises**

**QiskitError** – parameter errors

<span id="undefined" />

`mcry(theta, q_controls, q_target, q_ancillae, mode=None, use_basis_gates=False)`

Apply Multiple-Controlled Y rotation gate

**Parameters**

*   **self** ([*QuantumCircuit*](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")) – The QuantumCircuit object to apply the mcry gate on.
*   **theta** (*float*) – angle theta
*   **q\_controls** (*list(*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*)*) – The list of control qubits
*   **q\_target** ([*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")) – The target qubit
*   **q\_ancillae** ([*QuantumRegister*](qiskit.circuit.QuantumRegister#qiskit.circuit.QuantumRegister "qiskit.circuit.QuantumRegister") *or tuple(*[*QuantumRegister*](qiskit.circuit.QuantumRegister#qiskit.circuit.QuantumRegister "qiskit.circuit.QuantumRegister")*, int)*) – The list of ancillary qubits.
*   **mode** (*string*) – The implementation mode to use
*   **use\_basis\_gates** (*bool*) – use p, u, cx

**Raises**

**QiskitError** – parameter errors

<span id="undefined" />

`mcrz(lam, q_controls, q_target, use_basis_gates=False)`

Apply Multiple-Controlled Z rotation gate

**Parameters**

*   **self** ([*QuantumCircuit*](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")) – The QuantumCircuit object to apply the mcrz gate on.
*   **lam** (*float*) – angle lambda
*   **q\_controls** (*list(*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*)*) – The list of control qubits
*   **q\_target** ([*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")) – The target qubit
*   **use\_basis\_gates** (*bool*) – use p, u, cx

**Raises**

**QiskitError** – parameter errors

<span id="undefined" />

`mct(control_qubits, target_qubit, ancilla_qubits=None, mode='noancilla')`

Apply [`MCXGate`](qiskit.circuit.library.MCXGate#qiskit.circuit.library.MCXGate "qiskit.circuit.library.MCXGate").

<span id="undefined" />

`mcu1(lam, control_qubits, target_qubit)`

Apply `MCU1Gate`.

<span id="undefined" />

`mcx(control_qubits, target_qubit, ancilla_qubits=None, mode='noancilla')`

Apply [`MCXGate`](qiskit.circuit.library.MCXGate#qiskit.circuit.library.MCXGate "qiskit.circuit.library.MCXGate").

The multi-cX gate can be implemented using different techniques, which use different numbers of ancilla qubits and have varying circuit depth. These modes are: - ‘no-ancilla’: Requires 0 ancilla qubits. - ‘recursion’: Requires 1 ancilla qubit if more than 4 controls are used, otherwise 0. - ‘v-chain’: Requires 2 less ancillas than the number of control qubits. - ‘v-chain-dirty’: Same as for the clean ancillas (but the circuit will be longer).

<span id="undefined" />

`measure(qubit, cbit)`

Measure quantum bit into classical bit (tuples).

**Parameters**

*   **qubit** (*QuantumRegister|list|tuple*) – quantum register
*   **cbit** (*ClassicalRegister|list|tuple*) – classical register

**Returns**

the attached measure instruction.

**Return type**

qiskit.Instruction

**Raises**

**CircuitError** – if qubit is not in this circuit or bad format; if cbit is not in this circuit or not creg.

<span id="undefined" />

`measure_active(inplace=True)`

Adds measurement to all non-idle qubits. Creates a new ClassicalRegister with a size equal to the number of non-idle qubits being measured.

Returns a new circuit with measurements if inplace=False.

**Parameters**

**inplace** (*bool*) – All measurements inplace or return new circuit.

**Returns**

Returns circuit with measurements when inplace = False.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`measure_all(inplace=True)`

Adds measurement to all qubits. Creates a new ClassicalRegister with a size equal to the number of qubits being measured.

Returns a new circuit with measurements if inplace=False.

**Parameters**

**inplace** (*bool*) – All measurements inplace or return new circuit.

**Returns**

Returns circuit with measurements when inplace = False.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`mirror()`

DEPRECATED: use circuit.reverse\_ops().

**Returns**

the reversed circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`ms(theta, qubits)`

Apply [`MSGate`](qiskit.circuit.library.MSGate#qiskit.circuit.library.MSGate "qiskit.circuit.library.MSGate").

<span id="undefined" />

`property num_ancillas`

Return the number of ancilla qubits.

<span id="undefined" />

`property num_clbits`

Return number of classical bits.

<span id="undefined" />

`num_connected_components(unitary_only=False)`

How many non-entangled subcircuits can the circuit be factored to.

**Parameters**

**unitary\_only** (*bool*) – Compute only unitary part of graph.

**Returns**

Number of connected components in circuit.

**Return type**

int

<span id="undefined" />

`property num_layers`

Return the number of layers in the n-local circuit.

**Return type**

`int`

**Returns**

The number of layers in the circuit.

<span id="undefined" />

`num_nonlocal_gates()`

Return number of non-local gates (i.e. involving 2+ qubits).

Conditional nonlocal gates are also included.

<span id="undefined" />

`property num_parameters`

Convenience function to get the number of parameter objects in the circuit.

<span id="undefined" />

`property num_parameters_settable`

The number of total parameters that can be set to distinct values.

This does not change when the parameters are bound or exchanged for same parameters, and therefore is different from `num_parameters` which counts the number of unique [`Parameter`](qiskit.circuit.Parameter#qiskit.circuit.Parameter "qiskit.circuit.Parameter") objects currently in the circuit.

**Return type**

`int`

**Returns**

The number of parameters originally available in the circuit.

<Admonition title="Note" type="note">
  This quantity does not require the circuit to be built yet.
</Admonition>

<span id="undefined" />

`property num_qubits`

Returns the number of qubits in this circuit.

**Return type**

`int`

**Returns**

The number of qubits.

<span id="undefined" />

`num_tensor_factors()`

Computes the number of tensor factors in the unitary (quantum) part of the circuit only.

## Notes

This is here for backwards compatibility, and will be removed in a future release of Qiskit. You should call num\_unitary\_factors instead.

<span id="undefined" />

`num_unitary_factors()`

Computes the number of tensor factors in the unitary (quantum) part of the circuit only.

<span id="undefined" />

`property ordered_parameters`

The parameters used in the underlying circuit.

This includes float values and duplicates.

## Examples

```python
>>> # prepare circuit ...
>>> print(nlocal)
     ┌───────┐┌──────────┐┌──────────┐┌──────────┐
q_0: ┤ Ry(1) ├┤ Ry(θ[1]) ├┤ Ry(θ[1]) ├┤ Ry(θ[3]) ├
     └───────┘└──────────┘└──────────┘└──────────┘
>>> nlocal.parameters
{Parameter(θ[1]), Parameter(θ[3])}
>>> nlocal.ordered_parameters
[1, Parameter(θ[1]), Parameter(θ[1]), Parameter(θ[3])]
```

**Return type**

`List`\[`Parameter`]

**Returns**

The parameters objects used in the circuit.

<span id="undefined" />

`p(theta, qubit)`

Apply [`PhaseGate`](qiskit.circuit.library.PhaseGate#qiskit.circuit.library.PhaseGate "qiskit.circuit.library.PhaseGate").

<span id="undefined" />

`property parameter_bounds`

The parameter bounds for the unbound parameters in the circuit.

**Return type**

`Optional`\[`List`\[`Tuple`\[`float`, `float`]]]

**Returns**

A list of pairs indicating the bounds, as (lower, upper). None indicates an unbounded parameter in the corresponding direction. If None is returned, problem is fully unbounded.

<span id="undefined" />

`property parameters`

Get the [`Parameter`](qiskit.circuit.Parameter#qiskit.circuit.Parameter "qiskit.circuit.Parameter") objects in the circuit.

**Return type**

`Set`\[`Parameter`]

**Returns**

A set containing the unbound circuit parameters.

<span id="undefined" />

`power(power, matrix_power=False)`

Raise this circuit to the power of `power`.

If `power` is a positive integer and `matrix_power` is `False`, this implementation defaults to calling `repeat`. Otherwise, if the circuit is unitary, the matrix is computed to calculate the matrix power.

**Parameters**

*   **power** (*int*) – The power to raise this circuit to.
*   **matrix\_power** (*bool*) – If True, the circuit is converted to a matrix and then the matrix power is computed. If False, and `power` is a positive integer, the implementation defaults to `repeat`.

**Raises**

**CircuitError** – If the circuit needs to be converted to a gate but it is not unitary.

**Returns**

A circuit implementing this circuit raised to the power of `power`.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`property preferred_init_points`

The initial points for the parameters. Can be stored as initial guess in optimization.

**Return type**

`Optional`\[`List`\[`float`]]

**Returns**

The initial values for the parameters, or None, if none have been set.

<span id="undefined" />

`print_settings()`

Returns information about the setting.

**Return type**

`str`

**Returns**

The class name and the attributes/parameters of the instance as `str`.

<span id="undefined" />

`qasm(formatted=False, filename=None)`

Return OpenQASM string.

**Parameters**

*   **formatted** (*bool*) – Return formatted Qasm string.
*   **filename** (*str*) – Save Qasm to file with name ‘filename’.

**Returns**

If formatted=False.

**Return type**

str

**Raises**

**ImportError** – If pygments is not installed and `formatted` is `True`.

<span id="undefined" />

`qbit_argument_conversion(qubit_representation)`

Converts several qubit representations (such as indexes, range, etc.) into a list of qubits.

**Parameters**

**qubit\_representation** (*Object*) – representation to expand

**Returns**

Where each tuple is a qubit.

**Return type**

List(tuple)

<span id="undefined" />

`property qregs`

A list of the quantum registers associated with the circuit.

<span id="undefined" />

`qubit_duration(*qubits)`

Return the duration between the start and stop time of the first and last instructions, excluding delays, over the supplied qubits. Its time unit is `self.unit`.

**Parameters**

**\*qubits** – Qubits within `self` to include.

**Return type**

`Union`\[`int`, `float`]

**Returns**

Return the duration between the first start and last stop time of non-delay instructions

<span id="undefined" />

`qubit_start_time(*qubits)`

Return the start time of the first instruction, excluding delays, over the supplied qubits. Its time unit is `self.unit`.

Return 0 if there are no instructions over qubits

**Parameters**

*   **\*qubits** – Qubits within `self` to include. Integers are allowed for qubits, indicating
*   **of self.qubits.** (*indices*) –

**Return type**

`Union`\[`int`, `float`]

**Returns**

Return the start time of the first instruction, excluding delays, over the qubits

**Raises**

**CircuitError** – if `self` is a not-yet scheduled circuit.

<span id="undefined" />

`qubit_stop_time(*qubits)`

Return the stop time of the last instruction, excluding delays, over the supplied qubits. Its time unit is `self.unit`.

Return 0 if there are no instructions over qubits

**Parameters**

*   **\*qubits** – Qubits within `self` to include. Integers are allowed for qubits, indicating
*   **of self.qubits.** (*indices*) –

**Return type**

`Union`\[`int`, `float`]

**Returns**

Return the stop time of the last instruction, excluding delays, over the qubits

**Raises**

**CircuitError** – if `self` is a not-yet scheduled circuit.

<span id="undefined" />

`property qubits`

Returns a list of quantum bits in the order that the registers were added.

<span id="undefined" />

`r(theta, phi, qubit)`

Apply `RGate`.

<span id="undefined" />

`rcccx(control_qubit1, control_qubit2, control_qubit3, target_qubit)`

Apply [`RC3XGate`](qiskit.circuit.library.RC3XGate#qiskit.circuit.library.RC3XGate "qiskit.circuit.library.RC3XGate").

<span id="undefined" />

`rccx(control_qubit1, control_qubit2, target_qubit)`

Apply [`RCCXGate`](qiskit.circuit.library.RCCXGate#qiskit.circuit.library.RCCXGate "qiskit.circuit.library.RCCXGate").

<span id="undefined" />

`remove_final_measurements(inplace=True)`

Removes final measurement on all qubits if they are present. Deletes the ClassicalRegister that was used to store the values from these measurements if it is idle.

Returns a new circuit without measurements if inplace=False.

**Parameters**

**inplace** (*bool*) – All measurements removed inplace or return new circuit.

**Returns**

Returns circuit with measurements removed when inplace = False.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`repeat(reps)`

Repeat this circuit `reps` times.

**Parameters**

**reps** (*int*) – How often this circuit should be repeated.

**Returns**

A circuit containing `reps` repetitions of this circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

<span id="undefined" />

`property reps`

The number of times rotation and entanglement block are repeated.

**Return type**

`int`

**Returns**

The number of repetitions.

<span id="undefined" />

`reset(qubit)`

Reset q.

<span id="undefined" />

`reverse_bits()`

Return a circuit with the opposite order of wires.

The circuit is “vertically” flipped. If a circuit is defined over multiple registers, the resulting circuit will have the same registers but with their order flipped.

This method is useful for converting a circuit written in little-endian convention to the big-endian equivalent, and vice versa.

**Returns**

the circuit with reversed bit order.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

## Examples

**input:**

┌───┐

**q\_0: ┤ H ├─────■──────**

└───┘┌────┴─────┐

**q\_1: ─────┤ RX(1.57) ├**

└──────────┘

**output:**

┌──────────┐

**q\_0: ─────┤ RX(1.57) ├**

┌───┐└────┬─────┘

**q\_1: ┤ H ├─────■──────**

└───┘

<span id="undefined" />

`reverse_ops()`

Reverse the circuit by reversing the order of instructions.

This is done by recursively reversing all instructions. It does not invert (adjoint) any gate.

**Returns**

the reversed circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

## Examples

**input:**

┌───┐

**q\_0: ┤ H ├─────■──────**

└───┘┌────┴─────┐

**q\_1: ─────┤ RX(1.57) ├**

└──────────┘

**output:**

┌───┐

**q\_0: ─────■──────┤ H ├**

┌────┴─────┐└───┘

**q\_1: ┤ RX(1.57) ├─────**

└──────────┘

<span id="undefined" />

`property rotation_blocks`

The blocks in the rotation layers.

**Return type**

`List`\[`Instruction`]

**Returns**

The blocks in the rotation layers.

<span id="undefined" />

`rx(theta, qubit, label=None)`

Apply [`RXGate`](qiskit.circuit.library.RXGate#qiskit.circuit.library.RXGate "qiskit.circuit.library.RXGate").

<span id="undefined" />

`rxx(theta, qubit1, qubit2)`

Apply [`RXXGate`](qiskit.circuit.library.RXXGate#qiskit.circuit.library.RXXGate "qiskit.circuit.library.RXXGate").

<span id="undefined" />

`ry(theta, qubit, label=None)`

Apply [`RYGate`](qiskit.circuit.library.RYGate#qiskit.circuit.library.RYGate "qiskit.circuit.library.RYGate").

<span id="undefined" />

`ryy(theta, qubit1, qubit2)`

Apply [`RYYGate`](qiskit.circuit.library.RYYGate#qiskit.circuit.library.RYYGate "qiskit.circuit.library.RYYGate").

<span id="undefined" />

`rz(phi, qubit)`

Apply [`RZGate`](qiskit.circuit.library.RZGate#qiskit.circuit.library.RZGate "qiskit.circuit.library.RZGate").

<span id="undefined" />

`rzx(theta, qubit1, qubit2)`

Apply [`RZXGate`](qiskit.circuit.library.RZXGate#qiskit.circuit.library.RZXGate "qiskit.circuit.library.RZXGate").

<span id="undefined" />

`rzz(theta, qubit1, qubit2)`

Apply [`RZZGate`](qiskit.circuit.library.RZZGate#qiskit.circuit.library.RZZGate "qiskit.circuit.library.RZZGate").

<span id="undefined" />

`s(qubit)`

Apply [`SGate`](qiskit.circuit.library.SGate#qiskit.circuit.library.SGate "qiskit.circuit.library.SGate").

<span id="undefined" />

`sdg(qubit)`

Apply [`SdgGate`](qiskit.circuit.library.SdgGate#qiskit.circuit.library.SdgGate "qiskit.circuit.library.SdgGate").

<span id="undefined" />

`size()`

Returns total number of gate operations in circuit.

**Returns**

Total number of gate operations.

**Return type**

int

<span id="undefined" />

`snapshot(label, snapshot_type='statevector', qubits=None, params=None)`

Take a statevector snapshot of the internal simulator representation. Works on all qubits, and prevents reordering (like barrier). :param label: a snapshot label to report the result :type label: str :param snapshot\_type: the type of the snapshot. :type snapshot\_type: str :param qubits: the qubits to apply snapshot to \[Default: None]. :type qubits: list or None :param params: the parameters for snapshot\_type \[Default: None]. :type params: list or None

**Returns**

with attached command

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**ExtensionError** – malformed command

<span id="undefined" />

`snapshot_density_matrix(label, qubits=None)`

Take a density matrix snapshot of simulator state.

**Parameters**

*   **label** (*str*) – a snapshot label to report the result
*   **qubits** (*list or None*) – the qubits to apply snapshot to. If None all qubits will be snapshot \[Default: None].

**Returns**

with attached instruction.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**ExtensionError** – if snapshot is invalid.

<span id="undefined" />

`snapshot_expectation_value(label, op, qubits, single_shot=False, variance=False)`

Take a snapshot of expectation value \<O> of an Operator.

**Parameters**

*   **label** (*str*) – a snapshot label to report the result
*   **op** ([*Operator*](qiskit.quantum_info.Operator#qiskit.quantum_info.Operator "qiskit.quantum_info.Operator")) – operator to snapshot
*   **qubits** (*list*) – the qubits to snapshot.
*   **single\_shot** (*bool*) – return list for each shot rather than average \[Default: False]
*   **variance** (*bool*) – compute variance of values \[Default: False]

**Returns**

with attached instruction.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**ExtensionError** – if snapshot is invalid.

<span id="undefined" />

`snapshot_probabilities(label, qubits, variance=False)`

Take a probability snapshot of the simulator state.

**Parameters**

*   **label** (*str*) – a snapshot label to report the result
*   **qubits** (*list*) – the qubits to snapshot.
*   **variance** (*bool*) – compute variance of probabilities \[Default: False]

**Returns**

with attached instruction.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**ExtensionError** – if snapshot is invalid.

<span id="undefined" />

`snapshot_stabilizer(label)`

Take a stabilizer snapshot of the simulator state.

**Parameters**

**label** (*str*) – a snapshot label to report the result.

**Returns**

with attached instruction.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**ExtensionError** – if snapshot is invalid.

**Additional Information:**

This snapshot is always performed on all qubits in a circuit. The number of qubits parameter specifies the size of the instruction as a barrier and should be set to the number of qubits in the circuit.

<span id="undefined" />

`snapshot_statevector(label)`

Take a statevector snapshot of the simulator state.

**Parameters**

**label** (*str*) – a snapshot label to report the result.

**Returns**

with attached instruction.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**ExtensionError** – if snapshot is invalid.

**Additional Information:**

This snapshot is always performed on all qubits in a circuit. The number of qubits parameter specifies the size of the instruction as a barrier and should be set to the number of qubits in the circuit.

<span id="undefined" />

`squ(unitary_matrix, qubit, mode='ZYZ', up_to_diagonal=False, *, u=None)`

Decompose an arbitrary 2\*2 unitary into three rotation gates.

Note that the decomposition is up to a global phase shift. (This is a well known decomposition, which can be found for example in Nielsen and Chuang’s book “Quantum computation and quantum information”.)

**Parameters**

*   **unitary\_matrix** (*ndarray*) – 2\*2 unitary (given as a (complex) ndarray).
*   **qubit** (*QuantumRegister | Qubit*) – The qubit which the gate is acting on.
*   **mode** (*string*) – determines the used decomposition by providing the rotation axes. The allowed modes are: “ZYZ” (default)
*   **up\_to\_diagonal** (*bool*) – if set to True, the single-qubit unitary is decomposed up to a diagonal matrix, i.e. a unitary u’ is implemented such that there exists a 2\*2 diagonal gate d with u = d.dot(u’)
*   **u** (*ndarray*) – Deprecated, use `unitary_matrix` instead.

**Returns**

The single-qubit unitary instruction attached to the circuit.

**Return type**

[InstructionSet](qiskit.circuit.InstructionSet#qiskit.circuit.InstructionSet "qiskit.circuit.InstructionSet")

**Raises**

**QiskitError** – if the format is wrong; if the array u is not unitary

<span id="undefined" />

`swap(qubit1, qubit2)`

Apply [`SwapGate`](qiskit.circuit.library.SwapGate#qiskit.circuit.library.SwapGate "qiskit.circuit.library.SwapGate").

<span id="undefined" />

`sx(qubit)`

Apply [`SXGate`](qiskit.circuit.library.SXGate#qiskit.circuit.library.SXGate "qiskit.circuit.library.SXGate").

<span id="undefined" />

`sxdg(qubit)`

Apply [`SXdgGate`](qiskit.circuit.library.SXdgGate#qiskit.circuit.library.SXdgGate "qiskit.circuit.library.SXdgGate").

<span id="undefined" />

`t(qubit)`

Apply [`TGate`](qiskit.circuit.library.TGate#qiskit.circuit.library.TGate "qiskit.circuit.library.TGate").

<span id="undefined" />

`tdg(qubit)`

Apply [`TdgGate`](qiskit.circuit.library.TdgGate#qiskit.circuit.library.TdgGate "qiskit.circuit.library.TdgGate").

<span id="undefined" />

`to_gate(parameter_map=None, label=None)`

Create a Gate out of this circuit.

**Parameters**

*   **parameter\_map** (*dict*) – For parameterized circuits, a mapping from parameters in the circuit to parameters to be used in the gate. If None, existing circuit parameters will also parameterize the gate.
*   **label** (*str*) – Optional gate label.

**Returns**

a composite gate encapsulating this circuit (can be decomposed back)

**Return type**

[Gate](qiskit.circuit.Gate#qiskit.circuit.Gate "qiskit.circuit.Gate")

<span id="undefined" />

`to_instruction(parameter_map=None)`

Create an Instruction out of this circuit.

**Parameters**

**parameter\_map** (*dict*) – For parameterized circuits, a mapping from parameters in the circuit to parameters to be used in the instruction. If None, existing circuit parameters will also parameterize the instruction.

**Returns**

a composite instruction encapsulating this circuit (can be decomposed back)

**Return type**

[qiskit.circuit.Instruction](qiskit.circuit.Instruction#qiskit.circuit.Instruction "qiskit.circuit.Instruction")

<span id="undefined" />

`toffoli(control_qubit1, control_qubit2, target_qubit)`

Apply [`CCXGate`](qiskit.circuit.library.CCXGate#qiskit.circuit.library.CCXGate "qiskit.circuit.library.CCXGate").

<span id="undefined" />

`u(theta, phi, lam, qubit)`

Apply [`UGate`](qiskit.circuit.library.UGate#qiskit.circuit.library.UGate "qiskit.circuit.library.UGate").

<span id="undefined" />

`u1(theta, qubit)`

Apply [`U1Gate`](qiskit.circuit.library.U1Gate#qiskit.circuit.library.U1Gate "qiskit.circuit.library.U1Gate").

<span id="undefined" />

`u2(phi, lam, qubit)`

Apply [`U2Gate`](qiskit.circuit.library.U2Gate#qiskit.circuit.library.U2Gate "qiskit.circuit.library.U2Gate").

<span id="undefined" />

`u3(theta, phi, lam, qubit)`

Apply [`U3Gate`](qiskit.circuit.library.U3Gate#qiskit.circuit.library.U3Gate "qiskit.circuit.library.U3Gate").

<span id="undefined" />

`uc(gate_list, q_controls, q_target, up_to_diagonal=False)`

Attach a uniformly controlled gates (also called multiplexed gates) to a circuit.

The decomposition was introduced by Bergholm et al. in [https://arxiv.org/pdf/quant-ph/0410066.pdf](https://arxiv.org/pdf/quant-ph/0410066.pdf).

**Parameters**

*   **gate\_list** (*list\[ndarray]*) – list of two qubit unitaries \[U\_0,…,U\_\{2^k-1}], where each single-qubit unitary U\_i is a given as a 2\*2 array
*   **q\_controls** (*QuantumRegister|list\[(*[*QuantumRegister*](qiskit.circuit.QuantumRegister#qiskit.circuit.QuantumRegister "qiskit.circuit.QuantumRegister")*,int)]*) – list of k control qubits. The qubits are ordered according to their significance in the computational basis. For example if q\_controls=\[q\[1],q\[2]] (with q = QuantumRegister(2)), the unitary U\_0 is performed if q\[1] and q\[2] are in the state zero, U\_1 is performed if q\[2] is in the state zero and q\[1] is in the state one, and so on
*   **q\_target** (*QuantumRegister|(*[*QuantumRegister*](qiskit.circuit.QuantumRegister#qiskit.circuit.QuantumRegister "qiskit.circuit.QuantumRegister")*,int)*) – target qubit, where we act on with the single-qubit gates.
*   **up\_to\_diagonal** (*bool*) – If set to True, the uniformly controlled gate is decomposed up to a diagonal gate, i.e. a unitary u’ is implemented such that there exists a diagonal gate d with u = d.dot(u’), where the unitary u describes the uniformly controlled gate

**Returns**

the uniformly controlled gate is attached to the circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the list number of control qubits does not correspond to the provided number of single-qubit unitaries; if an input is of the wrong type

<span id="undefined" />

`ucrx(angle_list, q_controls, q_target)`

Attach a uniformly controlled (also called multiplexed) Rx rotation gate to a circuit.

The decomposition is base on [https://arxiv.org/pdf/quant-ph/0406176.pdf](https://arxiv.org/pdf/quant-ph/0406176.pdf) by Shende et al.

**Parameters**

*   **angle\_list** (*list*) – list of (real) rotation angles $[a_0,...,a_{2^k-1}]$
*   **q\_controls** (*QuantumRegister|list*) – list of k control qubits (or empty list if no controls). The control qubits are ordered according to their significance in increasing order: For example if `q_controls=[q[0],q[1]]` (with `q = QuantumRegister(2)`), the rotation `Rx(a_0)` is performed if `q[0]` and `q[1]` are in the state zero, the rotation `Rx(a_1)` is performed if `q[0]` is in the state one and `q[1]` is in the state zero, and so on
*   **q\_target** (*QuantumRegister|Qubit*) – target qubit, where we act on with the single-qubit rotation gates

**Returns**

the uniformly controlled rotation gate is attached to the circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the list number of control qubits does not correspond to the provided number of single-qubit unitaries; if an input is of the wrong type

<span id="undefined" />

`ucry(angle_list, q_controls, q_target)`

Attach a uniformly controlled (also called multiplexed) Ry rotation gate to a circuit.

The decomposition is base on [https://arxiv.org/pdf/quant-ph/0406176.pdf](https://arxiv.org/pdf/quant-ph/0406176.pdf) by Shende et al.

**Parameters**

*   **angle\_list** (*list\[numbers*) – list of (real) rotation angles $[a_0,...,a_{2^k-1}]$
*   **q\_controls** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of k control qubits (or empty list if no controls). The control qubits are ordered according to their significance in increasing order: For example if `q_controls=[q[0],q[1]]` (with `q = QuantumRegister(2)`), the rotation `Ry(a_0)` is performed if `q[0]` and `q[1]` are in the state zero, the rotation `Ry(a_1)` is performed if `q[0]` is in the state one and `q[1]` is in the state zero, and so on
*   **q\_target** (*QuantumRegister|Qubit*) – target qubit, where we act on with the single-qubit rotation gates

**Returns**

the uniformly controlled rotation gate is attached to the circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the list number of control qubits does not correspond to the provided number of single-qubit unitaries; if an input is of the wrong type

<span id="undefined" />

`ucrz(angle_list, q_controls, q_target)`

Attach a uniformly controlled (also called multiplexed gates) Rz rotation gate to a circuit.

The decomposition is base on [https://arxiv.org/pdf/quant-ph/0406176.pdf](https://arxiv.org/pdf/quant-ph/0406176.pdf) by Shende et al.

**Parameters**

*   **angle\_list** (*list\[numbers*) – list of (real) rotation angles \[a\_0,…,a\_\{2^k-1}]
*   **q\_controls** (*QuantumRegister|list\[*[*Qubit*](qiskit.circuit.Qubit#qiskit.circuit.Qubit "qiskit.circuit.Qubit")*]*) – list of k control qubits (or empty list if no controls). The control qubits are ordered according to their significance in increasing order: For example if q\_controls=\[q\[1],q\[2]] (with q = QuantumRegister(2)), the rotation Rz(a\_0)is performed if q\[1] and q\[2] are in the state zero, the rotation Rz(a\_1) is performed if q\[1] is in the state one and q\[2] is in the state zero, and so on
*   **q\_target** (*QuantumRegister|Qubit*) – target qubit, where we act on with the single-qubit rotation gates

**Returns**

the uniformly controlled rotation gate is attached to the circuit.

**Return type**

[QuantumCircuit](qiskit.circuit.QuantumCircuit#qiskit.circuit.QuantumCircuit "qiskit.circuit.QuantumCircuit")

**Raises**

**QiskitError** – if the list number of control qubits does not correspond to the provided number of single-qubit unitaries; if an input is of the wrong type

<span id="undefined" />

`unitary(obj, qubits, label=None)`

Apply unitary gate to q.

<span id="undefined" />

`width()`

Return number of qubits plus clbits in circuit.

**Returns**

Width of circuit.

**Return type**

int

<span id="undefined" />

`x(qubit, label=None)`

Apply [`XGate`](qiskit.circuit.library.XGate#qiskit.circuit.library.XGate "qiskit.circuit.library.XGate").

<span id="undefined" />

`y(qubit)`

Apply [`YGate`](qiskit.circuit.library.YGate#qiskit.circuit.library.YGate "qiskit.circuit.library.YGate").

<span id="undefined" />

`z(qubit)`

Apply [`ZGate`](qiskit.circuit.library.ZGate#qiskit.circuit.library.ZGate "qiskit.circuit.library.ZGate").