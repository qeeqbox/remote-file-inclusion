<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/remote-file-inclusion.svg"></p>

## Remote File Inclusion (RFI)
Remote File Inclusion (RFI) is a web application vulnerability that occurs when an application includes and executes a file from a remote server based on untrusted user input. If an attacker can control the file path or URL used by a file inclusion function, the application may download and execute malicious code, potentially leading to Remote Code Execution (RCE) and complete server compromise.

RFI is most commonly associated with PHP applications configured to allow remote file inclusion through settings such as `allow_url_include`.

## How RFI Works
1. Identify Vulnerable Parameter: Attackers look for parameters in web applications that are used to include files, such as `include()`, `require()`, `include_once()`, or `require_once()` in PHP.
2. Craft Malicious URL: Attackers create a URL that points to an external file on a server they control.
3. Exploit the Vulnerability: The attacker sends the malicious URL to the vulnerable parameter, causing the web application to include and execute the external file.

## Impact of RFI
Successful RFI can lead to:
- Remote Code Execution (RCE): Attackers may execute arbitrary code on the web server.
- Server Compromise: Attackers gain full control of the vulnerable server.
- Data Theft: Sensitive information, including databases, configuration files, and credentials, may be accessed or stolen.
- Malware Installation: Attackers may install web shells, ransomware, cryptominers, or other malicious software.
- Privilege Escalation: If the web server has excessive privileges, attackers may obtain broader access to the operating system.
- Reputation and Financial Damage: Organizations may face service disruption, financial losses, regulatory penalties, and loss of customer trust.

## RFI Mitigation Strategies
To prevent RFI:
- Never Trust User Input for File Inclusion: Do not allow users to directly specify file names or paths. Use predefined mappings or allowlists instead.
- Disable Remote File Inclusion: The remote file inclusion feature should be disabled unless it is absolutely necessary.
- Validate User Input: If user input must be accepted, ensure it is validated against a predefined list of acceptable values. Avoid allowing arbitrary file names or URLs.
- Follow the Principle of Least Privilege: Configure the web server with only the permissions necessary for its intended functions. This practice limits potential damage if an attacker successfully exploits the application.
- Keep Software Updated: Regularly apply security updates to reduce exposure to known vulnerabilities.
- Use a Web Application Firewall (WAF): A WAF can help detect and block malicious requests aimed at exploiting file inclusion vulnerabilities.
- Perform Regular Security Testing: Conduct routine security assessments to identify insecure file inclusion before attackers can exploit it.

## RFI Example
Clone this current repo recursively
```sh
git clone --recurse-submodules https://github.com/qeeqbox/remote-file-inclusion
```
Run the webapp using Python
```sh
python3 remote-file-inclusion/vulnerable-web-app/webapp.py
```
Open the webapp in your browser 127.0.0.1:5142
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/1.png"></p>
Use Joe's default credentials (username: joe and password: joe) to log in 
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/2.png"></p>
Open the Storage tab in the developer tools for network requests, and click Pull System Info
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/3.png"></p>
There will be an external request to this file https://raw.githubusercontent.com/qeeqbox/vulnerable-web-app/refs/heads/main/external/fullsysinfo.py 
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/4.png"></p>
The webapp retrieved the file and executed it (You can review this in the response tab)
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/5.png"></p>
Right-click the request, choose Edit and Resend, change the link for the external file to whoami.py or any remote Python script, then hit Send.
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/6.png"></p>
The file will be executed (You can review this in the response tab)
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/7.png"></p>

## Code
When a user clicks on an external module link, Pull System Info, the link for the external module is editable by the user
```py
def do_POST(self):
    ...
    ...
    elif parsed_url.path == "/external" and "link" in post_request_data:
        self.send_content(200, [('Content-type', 'text/html')], self.run_external_module(post_request_data["link"][0]))
        return
    ...
    ...
```
The run_external_module() function retrieves the external file and executes it without verification
```py
def run_external_module(self,link=None):
    ret = b""
    if link != None:
        with suppress():
            Valid = False
            parsed = urllib_parse.urlparse(link)
            filename = path.basename(parsed.path)
            file_content = b""
            with request.urlopen(link) as response, open(f"{path.join(EXTERNAL_FOLDER,filename)}","wb") as f:
                file_content = response.read()
                try:
                    ast_parse(file_content)
                    f.write(file_content)
                    Valid = True
                except:
                    Valid = False
            if Valid:
                with Popen([executable,f"{path.join(EXTERNAL_FOLDER,filename)}"], stdout=PIPE, stderr=STDOUT, close_fds=True) as process:
                     ret = process.communicate()[0]
    return ret
```
