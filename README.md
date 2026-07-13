<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/remote-file-inclusion.svg"></p>

An application retrieves and executes files from a remote source without proper validation. This can be exploited by a threat actor who provides a malicious remote file, leading the application to execute code controlled by the attacker. This may result in unauthorized access, remote code execution, data theft, or a complete compromise of the affected system.

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
The run_external_module() function gets retrieved and executed without verification
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
