Documentation for the digital digital readout.

- Get the digital-dro branch
- Compile the DDRO configuration
- Flash to a Due

Due pinouts are numbered as they appear on the board.

	Pin | Signal | Description
	------|------------|---------|-------------
	2 | X step | T
	1 | **Interlock Opens** | Opening one or more interlock switches causes `Interlock_NC` to go HI (active). The following things happen simultaneously:
