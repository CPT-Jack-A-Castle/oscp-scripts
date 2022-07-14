# Command Line Utilities
Various command line utilites that can support hacktivities. 

## Random stuff from course materials worth noting
- Search various man pages for keyword
	- `man -k compression`
- Find any files modified in the last day and not owned by root
	- `find . -type f -mtime -1 -not -user root -exec ls -l "{}" \;`
- Find locations of files and directories
	- `sudo updatedb && locate sbd.exe`
- Search for command in your history & execute it
	- `CTRL + R` - `{command}` - Enter to execute
- Monitor apache2 log file in real time
	- `sudo tail -f /var/log/apache2/access.log -n1` 
- Monitor the most CPU intensive processes in real time
	- `ps aux | sort -nrk 3,3 | head -n 5`
- capture traffic only on port 80
	- `sudo tcpdump -i tun0 tcp port 80 -w capture.pcap`
- display only ACK and SYN packets from a pcap file
	- `sudo tcpdump -n "tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)" -r capture.pcap`
- grab unique .html files from a log file
	- `cat log.txt | grep -Eio '/\S+\.html' | rev | cut -d / -f 1 | rev | sort -fu`