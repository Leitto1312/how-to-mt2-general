### PySockAPI Module
It's a simple module that uses socket connection to send commands to a m2 server (via adminpage)

Usage:

```
#(-c or --command) send command
	./pysock.py -c "<command>"
#(-g or --get) get con-data from con-file (pysock_con.txt) and send a command
	./pysock.py -g -c "<command>"
#(-f or --file) send command from file (1.)
	./pysock.py -f "<file>"
#(-h or --help) help
	./pysock.py -h
#(-s or --set) set con-data to con-file (pysock_con.txt) and send a command (2.)
	./pysock.py -s "<host>:<port>:<pwd>"
#(-r or --raw) raw mode (3.)
	./pysock.py -r "<host:port> <adminpwd> <command>"
```

Examples:

```
1. # ./pysock.py -f "mysock_cmd.txt"
2. # ./pysock.py -s "123.456.78.90:13000:SHOWMETHEMONEY" -c "NOTICE 1;NOTICE 2;NOTICE 3"
3. # ./pysock.py -r "173.194.35.6:13003 SHOWMETHEMONEY NOTICE 1;NOTICE 2;USER_COUNT"
```

---------------------------------------------------------

#### IT TRANSLATE

PySockApi è un semplice programmino (per freebsd) che da la possibilità di usare l'adminpage direttamente dal proprio server
(a mo' di localhost) e combinarlo con la shell per fare qualcosa di più automatico, dinamico e sicuro.
(non testatelo su altri server perché sui syslog appare l'ip di chi lo usa)

Vi ricordo alcune semplici cose:
- La porta da usare è quella che nel CONFIG è espressa con "PORT:" e non "P2P_PORT:".
- Per impostare i dati dell'adminpage sempre da CONFIG si fa così:

	```
	adminpage_ip1: vostro_ip
	adminpage_password: vostra_password
	```
- Il protocollo per la connessione socket che ho usato non permette di connettersi agli host "localhost|127.0.0.1" quindi dovrete mettere un ip reale (e non locale) del server.

#### Particolari del PySock:
- Con questa versione potrete leggere il return da parte del server (in caso di USER_COUNT apparirà il numero dei player on)
- All'inizio dell'__init__ ci sono 2 variabili che indicano che host e che dati dell'adminpage usare di default
- Il file "pysock_cmd.txt" può essere impostato così:

	```
	NOTICE Uno!
	NOTICE Due!
	NOTICE Ecc!
	```

- Con il parametro -s/--set si impostano i dati d'accesso sul file "pysock_con.txt" e si richiamano con -g/--get

Varie prove:
```
#invio comando+salvataggio dati su pysock_con.txt
./pysockapi.py -s "123.456.78.90:13000:SHOWMETHEMONEY" -c "NOTICE Yeah!;NOTICE Yeah!!!"
#una volta creato il file pysock_con.txt basta fare così:
./pysockapi.py -g -c "NOTICE Yeah!"
#per caricare roba da un file (nel mentre creiamolo):
echo NOTICE Yeah! >> pysock_cmd.txt
./pysockapi.py -g -f "pysock_cmd.txt"
#possiamo incrociare più roba:
./pysockapi.py -g -f "pysock_cmd.txt" -c "NOTICE Yeah!;EVENT xmas_boom 1;USER_COUNT;"
#completamente manuale:
./pysockapi.py -r "123.456.78.90:13000 SHOWMETHEMONEY Yeah!"
#miscuglio:
./pysockapi.py -r "123.456.78.90:13000 SHOWMETHEMONEY Yeah!" -g -f "pysock_cmd.txt" -c "NOTICE Yeah!"
#ripetuto:
./pysockapi.py -g -c "NOTICE Yeah!" -c "NOTICE Yeah!" -c "NOTICE Yeah!" -c "NOTICE Yeah!"
```

Ricordo che questo tool serve unicamente per il vostro server! (gestire eventi, mettere rates, bloccare chat e tanto altro da riga di comando!)

#### Da quest
Ecco una funzione in lua per richiamar il tool da quest:

```lua
function game.send_cmd(host,port,key,cmd)
	if(key==nil)then
		key="SHOWMETHEMONEY"
	end
	if(host==nil)or(port==nil)or(cmd==nil)then
		return false
	end
	if(type(cmd)=="table")then
		cmd=join("\n@",cmd) --[[require declare join function]]
	end
	os.execute([[/root/pysockapi.py -m "]]..host..[[:]]..port..[[;@]]..key..[[\n@]]..cmd..[[\n"]])
end

--to use
game.send_cmd("123.123.123.123","13003",nil,"EVENT xmas_sock 1")
--or
game.send_cmd("123.123.123.123","13003",nil,{"NOTICE Yeah!","EVENT xmas_sock 1", "EVENT xmas_tree 4", etc, etc})
```
