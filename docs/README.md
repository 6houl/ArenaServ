# ArenaServ Documentation

## Overview

This documentation repository contains comprehensive research and technical knowledge for ArenaServ, an Endless Online server project derived from Etheos/EOSERV.

## Purpose

This knowledge base exists to:

1. **Document the existing system** - Understand what ArenaServ currently does
2. **Map client capabilities** - Know what the vanilla EO client can and cannot do
3. **Provide evidence-based development** - Enable future AI agents to make technically correct decisions
4. **Preserve institutional knowledge** - Prevent rediscovery of architecture and constraints

## Documentation Structure

### Core Documentation
- [PROJECT-OVERVIEW.md](PROJECT-OVERVIEW.md) - High-level project description
- [AI-DEVELOPMENT-RULES.md](AI-DEVELOPMENT-RULES.md) - Rules for AI-assisted development
- [UPSTREAM-LINEAGE.md](UPSTREAM-LINEAGE.md) - Project history and ancestry
- [BUILD-ENVIRONMENT.md](BUILD-ENVIRONMENT.md) - Build setup and requirements
- [ROADMAP.md](ROADMAP.md) - Development roadmap
- [GITHUB-WORKFLOW.md](GITHUB-WORKFLOW.md) - Issue and project management

### Technical Areas

#### [architecture/](architecture/)
System architecture, components, and design patterns

#### [protocol/](protocol/)
- [PACKET-CATALOG.md](protocol/PACKET-CATALOG.md) - Complete packet reference
- EO network protocol documentation
- Server-client communication patterns

#### [client/](client/)
- [CLIENT-CAPABILITY-MATRIX.md](client/CLIENT-CAPABILITY-MATRIX.md) - What the client can do
- [WINDOW-CAPABILITIES.md](client/WINDOW-CAPABILITIES.md) - Available UI surfaces
- Vanilla client constraints and capabilities

#### [data-files/](data-files/)
- EMF (Map) format and capabilities
- EIF (Item) format and capabilities
- ENF (NPC) format and capabilities
- ESF (Spell) format and capabilities
- ECF (Class) format and capabilities
- Configuration file formats

#### [arena/](arena/)
- [LEGACY-ARENA-ARCHITECTURE.md](arena/LEGACY-ARENA-ARCHITECTURE.md)
- [LEGACY-ARENA-FLOW.md](arena/LEGACY-ARENA-FLOW.md)
- [ARENA-STATE-MACHINE.md](arena/ARENA-STATE-MACHINE.md)
- [ARENA-MOVEMENT.md](arena/ARENA-MOVEMENT.md)
- [ARENA-COMBAT.md](arena/ARENA-COMBAT.md)
- [ARENA-PACKETS.md](arena/ARENA-PACKETS.md)
- [ARENA-FAILURE-MODES.md](arena/ARENA-FAILURE-MODES.md)
- [ARENA-CLIENT-SURFACES.md](arena/ARENA-CLIENT-SURFACES.md)

#### [research/](research/)
- [UNKNOWN-BEHAVIOR.md](research/UNKNOWN-BEHAVIOR.md) - Unconfirmed behaviors
- Investigation notes and hypotheses
- Experimental findings

#### [decisions/](decisions/)
Architecture Decision Records (ADRs) documenting key technical choices

## Evidence Classification

All documented behavior is classified as:

- **CONFIRMED** - Verified in source code or through testing
- **PROBABLE** - Strong evidence but not definitively proven
- **EXPERIMENTAL** - Based on experiments, needs validation
- **UNKNOWN** - Insufficient evidence to conclude
- **CLIENT-REQUIRED** - Requires client modification (confirmed impossible server-side)

## Feature Classification

Features are classified by implementation requirement:

- **SERVER-ONLY** - Can be implemented entirely server-side
- **DATA-ASSISTED** - Requires data file (EIF/ENF/etc.) modifications only
- **CLIENT-REQUIRED** - Requires client code changes
- **UNKNOWN** - Not yet investigated

## Contributing to Documentation

When adding documentation:

1. Cite sources (file, class, function, line numbers)
2. Include evidence classification
3. Cross-reference related documentation
4. Note confidence level
5. Document unknowns explicitly rather than guessing
6. Update this index

## Research Methodology

1. Start with ArenaServ source code
2. Cross-reference Etheos upstream
3. Compare with EOSERV base
4. Consult eo-protocol specifications
5. Examine EOLib structures
6. Analyze data file formats
7. Test against vanilla client behavior
8. Document uncertainties for controlled experiments

---

**Last Updated:** 2026-09-22  
**Status:** Active Development
