# CLI Command Reference

<!--
  Copyright 2018-2021 Cargill Incorporated
  Licensed under Creative Commons Attribution 4.0 International License
  https://creativecommons.org/licenses/by/4.0/
-->

## splinter CLI
The `splinter` command-line interface (CLI) provides a set of commands to
interact with Splinter components.

[`splinter`]({% link docs/0.5/references/cli/splinter.1.md %})
Command-line interface for Splinter

### Authorized Identity Management
[`splinter authid`]({% link docs/0.5/references/cli/splinter-authid.1.md %})
Provides authorized identity subcommands.

[`splinter authid create`]({% link
docs/0.5/references/cli/splinter-authid-create.1.md %})
Creates an authorized identity on a Splinter node.

[`splinter authid delete`]({% link
docs/0.5/references/cli/splinter-authid-delete.1.md %})
Deletes an authorized identity on a Splinter node.

[`splinter authid list`]({% link
docs/0.5/references/cli/splinter-authid-list.1.md %})
Lists the authorized identities on a Splinter node.

[`splinter authid show`]({% link
docs/0.5/references/cli/splinter-authid-show.1.md %})
Shows a specific authorized identity on a Splinter node.

[`splinter authid update`]({% link
docs/0.5/references/cli/splinter-authid-update.1.md %})
Updates an authorized identity on a Splinter node.

### Certificate Management
[`splinter cert`]({% link docs/0.5/references/cli/splinter-cert.1.md %})
Provides certificate management subcommands

[`splinter cert
generate`]({% link docs/0.5/references/cli/splinter-cert-generate.1.md %})
Generates test certificates and keys for running splinterd with TLS (in insecure
mode %})

### Circuit Management
[`splinter circuit`]({% link docs/0.5/references/cli/splinter-circuit.1.md %})
Provides circuit management functionality.

[`splinter circuit
list`]({% link docs/0.5/references/cli/splinter-circuit-list.1.md %})
Displays the existing circuits for this Splinter node.

[`splinter circuit
proposals`]({% link docs/0.5/references/cli/splinter-circuit-proposals.1.md %})
Lists the current circuit proposals.

[`splinter circuit
propose`]({% link docs/0.5/references/cli/splinter-circuit-propose.1.md %})
Propose that a new circuit is created

[`splinter circuit
show`]({% link docs/0.5/references/cli/splinter-circuit-show.1.md %})
Displays information about a circuit

[`splinter circuit
template`]({% link docs/0.5/references/cli/splinter-circuit-template.1.md %})
 Manage circuit templates

[`splinter circuit template arguments`]({% link
docs/0.5/references/cli/splinter-circuit-template-arguments.1.md %})
Displays the arguments defined in a circuit template

[`splinter circuit template
show`]({% link docs/0.5/references/cli/splinter-circuit-template-show.1.md %})
Displays the details of a circuit template

[`splinter circuit template
list`]({% link docs/0.5/references/cli/splinter-circuit-template-list.1.md %})
Displays all available circuit templates

[`splinter circuit
vote`]({% link docs/0.5/references/cli/splinter-circuit-vote.1.md %})
Submits a vote to accept or reject a circuit proposal

### Database Management Functions for Biome
[`splinter database`]({% link docs/0.5/references/cli/splinter-database.1.md %})
Provides database management functions for Biome

[`splinter database
migrate`]({% link docs/0.5/references/cli/splinter-database-migrate.1.md %})
Updates the Biome database for a new Splinter release

### Display information about node and network health
[`splinter health`]({% link docs/0.5/references/cli/splinter-health.1.md %})
Displays information about node and network health

[`splinter health
status`]({% link docs/0.5/references/cli/splinter-health-status.1.md %})
Displays information about a Splinter node

### Generates user and daemon keys for Splinter
[`splinter keygen`]({% link docs/0.5/references/cli/splinter-keygen.1.md %})
Generates user and daemon keys for Splinter

### Role management

[`splinter role`]({% link docs/0.5/references/cli/splinter-role.1.md %})
Provides role subcommands.

[`splinter role create`]({% link
docs/0.5/references/cli/splinter-role-create.1.md %})
Creates a role on a Splinter node.

[`splinter role delete`]({% link
docs/0.5/references/cli/splinter-role-delete.1.md %})
Deletes a role on a Splinter node.

[`splinter role list`]({% link docs/0.5/references/cli/splinter-role-list.1.md
%})
Lists the roles on a Splinter node.

[`splinter role show`]({% link docs/0.5/references/cli/splinter-role-show.1.md
%})
Shows a specific role on a Splinter node.

[`splinter role update`]({% link
docs/0.5/references/cli/splinter-role-update.1.md %})
Updates a role on a Splinter node.

## splinterd CLI

The `splinterd` command-line interface (CLI) provides the command for running
the Splinter daemon.

[`splinterd`]({% link docs/0.5/references/cli/splinterd.1.md %})
Starts the Splinter daemon

## scabbard CLI
The `scabbard` command-line interface (CLI) provides a set of commands to
interact with scabbard services.

[`scabbard`]({% link docs/0.5/references/cli/scabbard.1.md %})
Command-line interface for scabbard

### Smart Contract Management

[`scabbard contract`]({% link docs/0.5/references/cli/scabbard-contract.1.md %})
Provides contract management functionality

[`scabbard contract
list`]({% link docs/0.5/references/cli/scabbard-contract-list.1.md %})
Displays a scabbard service's smart contracts

[`scabbard contract
show`]({% link docs/0.5/references/cli/scabbard-contract-show.1.md %})
Displays the details of a scabbard smart contract

[`scabbard contract
upload`]({% link docs/0.5/references/cli/scabbard-contract-upload.1.md %})
Uploads a smart contract to scabbard

### Contract Registry Management

[`scabbard cr`]({% link docs/0.5/references/cli/scabbard-cr.1.md %})
Provides management of the Sabre contract registry

[`scabbard cr
create`]({% link docs/0.5/references/cli/scabbard-cr-create.1.md %})
Creates a Sabre contract registry

[`scabbard cr
delete`]({% link docs/0.5/references/cli/scabbard-cr-delete.1.md %})
Deletes a Sabre contract registry

[`scabbard cr
update`]({% link docs/0.5/references/cli/scabbard-cr-update.1.md %})
Updates the owners of a Sabre contract registry

### Smart Contract Execution

[`scabbard exec`]({% link docs/0.5/references/cli/scabbard-exec.1.md %})
Executes a Sabre smart contract

### Namespace Management

[`scabbard ns`]({% link docs/0.5/references/cli/scabbard-ns.1.md %})
Provides management of Sabre namespaces

[`scabbard ns
create`]({% link docs/0.5/references/cli/scabbard-ns-create.1.md %})
Creates a Sabre namespace

[`scabbard ns
delete`]({% link docs/0.5/references/cli/scabbard-ns-delete.1.md %})
Deletes a Sabre namespace

[`scabbard ns
update`]({% link docs/0.5/references/cli/scabbard-ns-update.1.md %})
Updates the owners of a Sabre namespace

### Namespace Permissions Management

[`scabbard perm`]({% link docs/0.5/references/cli/scabbard-perm.1.md %})
Sets or deletes a Sabre namespace permission

### Smart Permissions Management

[`scabbard sp`]({% link docs/0.5/references/cli/scabbard-sp.1.md %})
Provides management of Sabre smart permissions

[`scabbard sp
create`]({% link docs/0.5/references/cli/scabbard-sp-create.1.md %})
Creates a Sabre smart permission

[`scabbard sp
delete`]({% link docs/0.5/references/cli/scabbard-sp-delete.1.md %})
Deletes a Sabre smart permission

[`scabbard sp
update`]({% link docs/0.5/references/cli/scabbard-sp-update.1.md %})
Updates an existing Sabre smart permission
