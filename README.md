### The below python code use to pull the data from website (stackflow.py).

	import subprocess
	import mechanize
	import urlparse
	from bs4 import BeautifulSoup
	import urllib2 
	import cookielib
	cj = cookielib.CookieJar()
	br = mechanize.Browser()
	br.set_cookiejar(cj)
	br.open("https://******/login")

	br.select_form(nr=0)
	br.form['name'] = 'username'
	br.form['password'] = 'password'
	br.submit()
	#print =br.response().read()
	with open('contrat.txt') as f:
		for line in f:

			response = br.open(urlparse.urljoin('https://targetpage', line))
			r = response.read()
			text_file = open('output.txt', 'w')
			text_file.write(r)
			text_file.close()
			test = (subprocess.check_output("cat output.txt  | grep string1", shell=True))
			urls = re.findall(r'href=[\'"]?([^\'" >]+)', test) # we are finding all links in test
			con =  ', '.join(urls).split("/")[4] #4th link on page
			response1 = br.open(urlparse.urljoin('https://another url here', con)) #concatenation
			w =  response1.read()
			text_file = open('output1.txt', 'w')
			text_file.write(w)
			text_file.close()
			test1 = (subprocess.check_output("cat output1.txt | grep -w -A1 -e emp_name -e code_name -e address -e id | grep -v -e '<th>' -e 'label' -e 'input'", shell=True))
			print line
			print test1



### The below code will align the report, 

	python stackflow.py > 02_06.txt

	cat 02_06.txt  | sed 's/<\/td>//g' | sed 's/<td>//g' | sed 's/^[ \t]*//;s/[ \t]*$//' | grep -v -e ^- -e ^$ | grep -v -B1 ^econ | grep -v  ^- | sed "s/$/,/g"| awk 'ORS=NR%5?FS:RS'

	1)sed 's/<\/td>//g' - will remove the <\/td> tag
	2)sed 's/<td>//g' - will remove the <td> tag
	3)sed 's/^[ \t]*//;s/[ \t]*$//' - replaces leading whitespace &  trailing whitespace
	4)grep -v -e ^- -e ^$ - Remove line start with - and  blanks lines
	5)grep -v -B1 ^econ - remove one line before the matching pattern.
	6)grep -v  ^- - Remove line start with -
	7)sed "s/$/,/g" - Add comma at end of every lines.
	8)awk 'ORS=NR%5?FS:RS' - Concatenate 5 words per line.


### The below python script is use to auto login on jump server from local system and form tunnel with storage box without any addition package on any servers
### Install paramiko module on local machine.
### Run the below script from local machine.

	import paramiko
	from sshtunnel import SSHTunnelForwarder
	with SSHTunnelForwarder(
	   ('servername', 22),		#jump server address
	    ssh_username='ajay.prajapati',
	    ssh_pkey=paramiko.RSAKey.from_private_key_file("/root/.ssh/id_rsa"),
	    #ssh_private_key_password="*******",
	   remote_bind_address=("*.*.*.*", 22), #storage box ip address 
	   local_bind_address=('127.0.0.1', 10023)
	) as tunnel:
	   client = paramiko.SSHClient()
		#client.load_system_host_keys()
	   client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
	   client.connect(hostname='127.0.0.1', username="root", password="****", port=10023, look_for_keys=False)
		# do some operations with client session
	   stdin, stdout, stderr = (client.exec_command('./datapull.sh')) # it will run the datapull.sh which is on storage box.
	   print stdout.readlines()
	   print stderr.readlines()
	   client.close()

