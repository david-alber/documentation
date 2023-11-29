---
title: AccountProvider
description: API reference for qiskit.providers.ibmq.AccountProvider
in_page_toc_min_heading_level: 1
python_api_type: class
python_api_name: qiskit.providers.ibmq.AccountProvider
---

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

# AccountProvider[¶](#accountprovider "Permalink to this headline")

<span id="qiskit.providers.ibmq.AccountProvider" />

`AccountProvider(credentials, factory)`

Bases: [`qiskit.providers.provider.ProviderV1`](qiskit.providers.ProviderV1 "qiskit.providers.provider.ProviderV1")

Provider for a single IBM Quantum Experience account.

The account provider class provides access to the IBM Quantum Experience services available to this account.

You can access a provider by enabling an account with the [`IBMQ.enable_account()`](qiskit.providers.ibmq.IBMQFactory#enable_account "qiskit.providers.ibmq.IBMQFactory.enable_account") method, which returns the default provider you have access to:

```python
from qiskit import IBMQ
provider = IBMQ.enable_account(<INSERT_IBM_QUANTUM_EXPERIENCE_TOKEN>)
```

To select a different provider, use the [`IBMQ.get_provider()`](qiskit.providers.ibmq.IBMQFactory#get_provider "qiskit.providers.ibmq.IBMQFactory.get_provider") method and specify the hub, group, or project name of the desired provider.

Each provider may offer different services. The main service, [`IBMQBackendService`](qiskit.providers.ibmq.IBMQBackendService "qiskit.providers.ibmq.ibmqbackendservice.IBMQBackendService"), is available to all providers and gives access to IBM Quantum Experience devices and simulators.

You can obtain an instance of a service using the [`service()`](qiskit.providers.ibmq.AccountProvider#service "qiskit.providers.ibmq.AccountProvider.service") method or as an attribute of this `AccountProvider` instance. For example:

```python
backend_service = provider.service('backend')
backend_service = provider.service.backend
```

Since [`IBMQBackendService`](qiskit.providers.ibmq.IBMQBackendService "qiskit.providers.ibmq.ibmqbackendservice.IBMQBackendService") is the main service, some of the backend-related methods are available through this class for convenience.

The [`backends()`](qiskit.providers.ibmq.AccountProvider#backends "qiskit.providers.ibmq.AccountProvider.backends") method returns all the backends available to this account:

```python
backends = provider.backends()
```

The [`get_backend()`](qiskit.providers.ibmq.AccountProvider#get_backend "qiskit.providers.ibmq.AccountProvider.get_backend") method returns a backend that matches the filters passed as argument. An example of retrieving a backend that matches a specified name:

```python
simulator_backend = provider.get_backend('ibmq_qasm_simulator')
```

It is also possible to use the `backend` attribute to reference a backend. As an example, to retrieve the same backend from the example above:

```python
simulator_backend = provider.backend.ibmq_qasm_simulator
```

<Admonition title="Note" type="note">
  The `backend` attribute can be used to autocomplete the names of backends available to this provider. To autocomplete, press `tab` after `provider.backend.`. This feature may not be available if an error occurs during backend discovery. Also note that this feature is only available in interactive sessions, such as in Jupyter Notebook and the Python interpreter.
</Admonition>

AccountProvider constructor.

**Parameters**

*   **credentials** ([`Credentials`](qiskit.providers.ibmq.credentials.Credentials "qiskit.providers.ibmq.credentials.credentials.Credentials")) – IBM Quantum Experience credentials.
*   **factory** ([`IBMQFactory`](qiskit.providers.ibmq.IBMQFactory "qiskit.providers.ibmq.ibmqfactory.IBMQFactory")) – IBM Quantum account.

## Methods

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

### backends

<span id="qiskit.providers.ibmq.AccountProvider.backends" />

`AccountProvider.backends(name=None, filters=None, **kwargs)`

Return all backends accessible via this provider, subject to optional filtering.

**Parameters**

*   **name** (`Optional`\[`str`]) – Backend name to filter by.

*   **filters** (`Optional`\[`Callable`\[\[`List`\[[`IBMQBackend`](qiskit.providers.ibmq.IBMQBackend "qiskit.providers.ibmq.ibmqbackend.IBMQBackend")]], `bool`]]) –

    More complex filters, such as lambda functions. For example:

    ```python
    AccountProvider.backends(filters=lambda b: b.configuration().n_qubits > 5)
    ```

*   **kwargs** (`Any`) –

    Simple filters that specify a `True`/`False` criteria in the backend configuration, backends status, or provider credentials. An example to get the operational backends with 5 qubits:

    ```python
    AccountProvider.backends(n_qubits=5, operational=True)
    ```

**Return type**

`List`\[[`IBMQBackend`](qiskit.providers.ibmq.IBMQBackend "qiskit.providers.ibmq.ibmqbackend.IBMQBackend")]

**Returns**

The list of available backends that match the filter.

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

### get\_backend

<span id="qiskit.providers.ibmq.AccountProvider.get_backend" />

`AccountProvider.get_backend(name=None, **kwargs)`

Return a single backend matching the specified filtering.

**Parameters**

*   **name** (*str*) – name of the backend.
*   **\*\*kwargs** – dict used for filtering.

**Returns**

a backend matching the filtering.

**Return type**

[Backend](qiskit.providers.Backend "qiskit.providers.Backend")

**Raises**

[**QiskitBackendNotFoundError**](qiskit.providers.QiskitBackendNotFoundError "qiskit.providers.QiskitBackendNotFoundError") – if no backend could be found or more than one backend matches the filtering criteria.

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

### has\_service

<span id="qiskit.providers.ibmq.AccountProvider.has_service" />

`AccountProvider.has_service(name)`

Check if this provider has access to the service.

**Parameters**

**name** (`str`) – Name of the service.

**Return type**

`bool`

**Returns**

Whether the provider has access to the service.

**Raises**

**IBMQInputValueError** – If an unknown service name is specified.

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

### run\_circuits

<span id="qiskit.providers.ibmq.AccountProvider.run_circuits" />

`AccountProvider.run_circuits(circuits, backend_name, shots=None, initial_layout=None, layout_method=None, routing_method=None, translation_method=None, seed_transpiler=None, optimization_level=1, init_qubits=True, rep_delay=None, transpiler_options=None, measurement_error_mitigation=False, use_measure_esp=None, **run_config)`

Execute the input circuit(s) on a backend using the runtime service.

<Admonition title="Note" type="note">
  This method uses the IBM Quantum runtime service which is not available to all accounts.
</Admonition>

**Parameters**

*   **circuits** (`Union`\[[`QuantumCircuit`](qiskit.circuit.QuantumCircuit "qiskit.circuit.quantumcircuit.QuantumCircuit"), `List`\[[`QuantumCircuit`](qiskit.circuit.QuantumCircuit "qiskit.circuit.quantumcircuit.QuantumCircuit")]]) – Circuit(s) to execute.
*   **backend\_name** (`str`) – Name of the backend to execute circuits on. Transpiler options are automatically grabbed from backend configuration and properties unless otherwise specified.
*   **shots** (`Optional`\[`int`]) – Number of repetitions of each circuit, for sampling. If not specified, the backend default is used.
*   **initial\_layout** (`Union`\[[`Layout`](qiskit.transpiler.Layout "qiskit.transpiler.layout.Layout"), `Dict`, `List`, `None`]) – Initial position of virtual qubits on physical qubits.
*   **layout\_method** (`Optional`\[`str`]) – Name of layout selection pass (‘trivial’, ‘dense’, ‘noise\_adaptive’, ‘sabre’). Sometimes a perfect layout can be available in which case the layout\_method may not run.
*   **routing\_method** (`Optional`\[`str`]) – Name of routing pass (‘basic’, ‘lookahead’, ‘stochastic’, ‘sabre’)
*   **translation\_method** (`Optional`\[`str`]) – Name of translation pass (‘unroller’, ‘translator’, ‘synthesis’)
*   **seed\_transpiler** (`Optional`\[`int`]) – Sets random seed for the stochastic parts of the transpiler.
*   **optimization\_level** (`int`) – How much optimization to perform on the circuits. Higher levels generate more optimized circuits, at the expense of longer transpilation time. If None, level 1 will be chosen as default.
*   **init\_qubits** (`bool`) – Whether to reset the qubits to the ground state for each shot.
*   **rep\_delay** (`Optional`\[`float`]) – Delay between programs in seconds. Only supported on certain backends (`backend.configuration().dynamic_reprate_enabled` ). If supported, `rep_delay` will be used instead of `rep_time` and must be from the range supplied by the backend (`backend.configuration().rep_delay_range`). Default is given by `backend.configuration().default_rep_delay`.
*   **transpiler\_options** (`Optional`\[`dict`]) – Additional transpiler options.
*   **measurement\_error\_mitigation** (`bool`) – Whether to apply measurement error mitigation.
*   **use\_measure\_esp** (`Optional`\[`bool`]) – Whether to use excited state promoted (ESP) readout for measurements which are the final instruction on a qubit. ESP readout can offer higher fidelity than standard measurement sequences. See [here](https://arxiv.org/pdf/2008.08571.pdf).
*   **\*\*run\_config** – Extra arguments used to configure the circuit execution.

**Return type**

[`RuntimeJob`](qiskit.providers.ibmq.runtime.RuntimeJob "qiskit.providers.ibmq.runtime.runtime_job.RuntimeJob")

**Returns**

Runtime job.

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

### service

<span id="qiskit.providers.ibmq.AccountProvider.service" />

`AccountProvider.service(name)`

Return the specified service.

**Parameters**

**name** (`str`) – Name of the service.

**Return type**

`Any`

**Returns**

The specified service.

**Raises**

*   **IBMQInputValueError** – If an unknown service name is specified.
*   **IBMQNotAuthorizedError** – If the account is not authorized to use the service.

<Admonition title="Warning" type="caution">
  The package `qiskit-ibmq-provider` is being deprecated and its repo is going to be archived soon. Please transition to the new packages. More information in [https://ibm.biz/provider\_migration\_guide](https://ibm.biz/provider_migration_guide)
</Admonition>

### services

<span id="qiskit.providers.ibmq.AccountProvider.services" />

`AccountProvider.services()`

Return all available services.

**Return type**

`Dict`

**Returns**

All services available to this provider.

## Attributes

<span id="qiskit.providers.ibmq.AccountProvider.backend" />

### backend

Return the backend service.

**Return type**

[`IBMQBackendService`](qiskit.providers.ibmq.IBMQBackendService "qiskit.providers.ibmq.ibmqbackendservice.IBMQBackendService")

**Returns**

The backend service instance.

<span id="qiskit.providers.ibmq.AccountProvider.experiment" />

### experiment

Return the experiment service.

**Return type**

[`IBMExperimentService`](qiskit.providers.ibmq.experiment.IBMExperimentService "qiskit.providers.ibmq.experiment.ibm_experiment_service.IBMExperimentService")

**Returns**

The experiment service instance.

**Raises**

**IBMQNotAuthorizedError** – If the account is not authorized to use the experiment service.

<span id="qiskit.providers.ibmq.AccountProvider.runtime" />

### runtime

Return the runtime service.

**Return type**

[`IBMRuntimeService`](qiskit.providers.ibmq.runtime.IBMRuntimeService "qiskit.providers.ibmq.runtime.ibm_runtime_service.IBMRuntimeService")

**Returns**

The runtime service instance.

**Raises**

**IBMQNotAuthorizedError** – If the account is not authorized to use the service.

<span id="qiskit.providers.ibmq.AccountProvider.version" />

### version

`= 1`
