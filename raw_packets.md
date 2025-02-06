C++ source code for `SMLabeler` was already sent in July 2021.
This software loads a file that contains raw Bluetooth packets, parses the packets, analyzes them, and display analysis results such as heart rate, respiratory rate, movement, sleep stages etc.
The source code contains `parser` directory, where `DataParser` class does all data parsing.
Namely, `DataParser::parseBinFile` does the actual parsing.
- Each packet is 102 bytes long.
- The first 5 bytes must be 0xAA64440101.
- The 6th byte indicates the packet id. (0-127: Must be consecutive. Otherwise, the Bluetooth connection is not stable.)
- Packet body starts from the 7th byte and ends at the 96th byte (inclusive).
- 97th, 98th, 99th, and 102nd bytes must be 0x00, 0x00, 0x00, and 0xCC respectively.
- 100th, 101st bytes are packet checksum.
	checksum = 0x0153 + sum(6th byte to 96th byte)
	100th byte = 8 MSB of checksum
	101st byte = 8 LSB of checksum
- 30 data points for 24-bit ADC are stored in the packet body (3 x 30 = 90 bytes in total). But these 90 bytes are permutated to obfuscate the contents of the packet body.
	Pseudocode:
		real_body_byte[i] = received_body_byte[conversion_array[i]]; // 0-based indexing
		conversion_array[90]={17,82,38,29,72,64,31,48,23,75,55,40,2,85,60,9,44,59,68,0,35,58,83,33,45,11,87,69,52,46,74,8,19,26,77,30,56,10,24,53,27,32,81,28,78,13,6,84,70,34,66,89,5,20,37,71,88,61,47,16,86,42,39,14,4,79,50,18,1,80,54,21,36,65,41,3,25,62,43,22,7,49,73,51,63,76,15,67,12,57};
	e.g.: (Note that the above pseudocode is 0-based indexing, while the other parts of this text are 1-based indexing.)
		The first ADB measurement data (3 bytes) are (17+1)th, (82+1)th, and (38+1)th bytes of the received body
		The second are (29+1)th, (72+1)nd, and (64+1)th bytes
		etc
