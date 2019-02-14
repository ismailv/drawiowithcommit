<html>
<title>Diagrams for GitHub</title>
<style>
input, select {
  width: 100%;
  padding: 5px 5px;
  margin: 4px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

button {
  width: 60%;
  background-color: #730000;
  color: white;
  padding: 5px 5px;
  margin: 4px 0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

input[type=submit]:hover {
  background-color: #45a049;
}

div {
  border-radius: 5px;
  background-color: #f2f2f2;
  padding: 20px;
}
</style>
<head>
	<style type="text/css">
        iframe {
            border: 0;
            background-color: #99badd;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            width: 100%;
            height: 100%;
        }
	</style>
	<script type="text/javascript">
		var editor = 'https://www.draw.io/?embed=1&ui=atlas&spin=1&proto=json';

		var urlParams = (function()
		{
			var result = new Object();
			var params = window.location.search.slice(1).split('&');
			
			for (var i = 0; i < params.length; i++)
			{
				idx = params[i].indexOf('=');
				
				if (idx > 0)
				{
					result[params[i].substring(0, idx)] = params[i].substring(idx + 1);
				}
			}
			
			return result;
		})();

		function edit(path, content, callback)
		{
			var iframe = document.createElement('iframe');
			iframe.setAttribute('frameborder', '0');

			var close = function()
			{
				window.removeEventListener('message', receive);
				document.body.removeChild(iframe);
			};
			
			var save = function(data)
			{
				iframe.contentWindow.postMessage(JSON.stringify(
				{
					action: 'spinner',
					message: 'Saving',
					show: 1
				}), '*');
				
				callback(data, function(success)
				{
					iframe.contentWindow.postMessage(JSON.stringify(
					{
						action: 'spinner',
						show: 0
					}), '*');
				
					if (success)
					{
						close();
					}
				}, iframe);
			};

			var receive = function(evt)
			{
				if (evt.data.length > 0)
				{
					var msg = JSON.parse(evt.data);
					
					if (msg.event == 'init')
					{
						if (content == null)
						{
							iframe.contentWindow.postMessage(JSON.stringify(
							{
								action: 'load',
								autosave: 0,
								xml: ''
							}), '*');
						}
						else if (/(\.png)$/i.test(path))
						{
							iframe.contentWindow.postMessage(JSON.stringify(
							{
								action: 'load',
								autosave: 0,
								xmlpng: 'data:image/png;base64,' + content
							}), '*');
						}
						else
						{
							iframe.contentWindow.postMessage(JSON.stringify(
							{
								action: 'load',
								autosave: 0,
								xml: atob(content)
							}), '*');
						}
					}
					else if (msg.event == 'export')
					{
						var data = (msg.data.substring(0, 5) == 'data:') ?
							msg.data.substring(msg.data.indexOf(',') + 1) :
							btoa(msg.data);
						save(data);
					}
					else if (msg.event == 'save')
					{
						if ((/\.(png|svg|html)$/i).test(path))
						{
							var ext = path.substring(path.lastIndexOf('.') + 1);
						
							// Additional export step required for PNG, SVG and HTML
							iframe.contentWindow.postMessage(JSON.stringify(
							{
								spin: 'Saving',
								action: 'export',
								format: (/(\.html)$/i.test(path)) ? 'html' : 'xml' + ext,
								xml: msg.xml
							}), '*');
						}
						else
						{
							save(btoa(msg.xml));
						}
					}
					else if (msg.event == 'exit')
					{
						close();
					}
				}
			};

			window.addEventListener('message', receive);
			iframe.setAttribute('src', editor);
			document.body.appendChild(iframe);
		};
		
		function xhr(verb, url, data, callback)
		{
			var req = new XMLHttpRequest();
			req.onreadystatechange = function()
			{
				if (req.readyState == 4)
				{
					callback(req);
				}
			};
			
			req.open(verb, url, true);
			
			var username = document.getElementById('username').value;
			var pass = document.getElementById('password').value;
			req.setRequestHeader('Authorization', 'Basic ' +
				btoa(username + ':' + pass));
			
			req.send(data);
		}
		
		function readFile(sentProcess)
		{
			var button = document.getElementById('btnlist');
			var prev = button.innerHTML;

			button.setAttribute('disabled', 'disabled');
			button.innerHTML = 'Loading...';
			
			var org = document.getElementById('org').value;
			var repo = document.getElementById('repo').value;
			var path = document.getElementById('path').value;
			var ref = document.getElementById('ref').value;
			var url = 'https://api.github.com/repos/' + org + '/' + repo +
				'/contents/' + path + '?ref=' + encodeURIComponent(ref);
			
			xhr('GET', url, null, function(req)
			{
				button.removeAttribute('disabled');
				button.innerHTML = prev;
				
				var obj = JSON.parse(req.responseText);
				var filename = path;
				var slash = filename.lastIndexOf('/');
				
				if (slash >= 0)
				{
					filename = filename.substring(slash + 1);
				}
				console.log('sha', obj);
			
				if (req.status == 200 || req.status == 404)
				{
				    if (sentProcess=='p'){	
					edit(path, obj.content, function(data, fn, iframe)
					{
					    var msg = prompt('Commit Message', 'Update ' + filename);					
					    writeFile(url, path, obj.sha, msg || '', data, fn);
					 

					    
					});
				    } else {
				        console.log('sha1', obj);
				        var sel = document.getElementById('reposelect');
				        for (var i = 0; i < obj.length; i++) {
				            var opt = document.createElement('option');
				            opt.innerHTML = obj[i].name;
				            opt.value = obj[i].path;
				            sel.appendChild(opt);
				        }
				    }
				}
				else
				{
					alert(obj.message);
				}
			});
			
			xhr('GET', 'https://api.github.com/repos/'+ org + '/' + repo +  '/commits'  , null, function(req){
				console.log(req);
				 var selcom = document.getElementById('commitselect');
				 var jsonreturn = JSON.parse(req.responseText);
				        for (var i = 0; i < jsonreturn.length; i++) {
				            var opt = document.createElement('option');
				            opt.innerHTML = jsonreturn[i].commit.message;
				            opt.value = jsonreturn[i].sha;
				            selcom.appendChild(opt);
				        }
			
			}
			)
		};
		
		function writeFile(url, path, sha, msg, content, fn)
		{
		    console.log('sha', sha);
			var entity =
			{
                sha: sha,
			    committer: {
			        name: 'ismailv',
                    email: 'ismailv.akar@gmail.com'
			    },
				message: msg,
				content: content,
			};			
			
		
			xhr('PUT', url, JSON.stringify(entity), function(req)
			{
				if (req.readyState == 4)
				{
					var success = req.status == 200 || req.status == 201;
					fn(success);
					alert(msg + ' commitiniz başarı ile gönderildi');

					if (!success)
					{
					    var obj = JSON.parse(req.responseText);
					    console.log('obj', obj);
						alert(obj.message);
					}
				}
			});
		};
		
		function onload()
		{
			document.getElementById('username').value = urlParams['user'] || 'ismailv';
			document.getElementById('password').value = urlParams['pass'] || '';
			document.getElementById('org').value = urlParams['org'] || 'ismailv';
			document.getElementById('repo').value = urlParams['repo'] || '';
			document.getElementById('path').value = urlParams['path'] || '';
			document.getElementById('ref').value = urlParams['ref'] || 'master';
			
			document.getElementById((urlParams['username'] == null) ?
				'username' : ((urlParams['password'] == null) ?
				'password' : 'button')).focus();
				
			document.addEventListener('keydown', function(evt)
			{
				if (evt.keyCode == 13)
				{
					document.getElementById('button').click();
				}
			});
			
			if (urlParams['action'] == 'open')
			{
				document.getElementById('button').click();
			}
		};

		function getGitFiles() {
		    readFile('x');
		}

		function setPath(index) {
		    console.log('index', index);
		    var e = document.getElementById("reposelect");
		    var strValue = e.options[index].value;
		    console.log('value', strValue);
		    document.getElementById('path').value = strValue;
		}

	</script>
</head>
<body onload="onload();">
<h2>AsyncDiagrams (with GitHub)</h2>
<table>
<tr><td>Git Kullanıcı Adın:</td><td><input id="username" type="text"/></td></tr>
<tr><td>Git Şifren:</td><td><input id="password" type="password"/></td></tr>
<tr><td>Organization:</td><td><input id="org" type="org"/></td></tr>
<tr><td>Repository:</td><td><input id="repo" type="text"/></td></tr>
<tr><td>Seçtiğin Dosya:</td><td><select id="reposelect" onchange="setPath(this.selectedIndex);"><option></option></select><button id="btnlist" onclick="getGitFiles();" style="margin-left:2.5rem;">Listele</button></td></tr>
<tr><td>Commit Seçiniz:</td><td><select id="commitselect" "><option></option></select></td></tr>

<tr><td>Path:</td><td><input id="path"type="text" disabled/></td></tr>
<tr><td>Ref:</td><td><input id="ref" type="text"/></td></tr>
<tr><td colspan="2" align="right">
<br>
<button id="button" onclick="readFile('p');">Ekle/Yarat</button>
</td></tr>
</table>
<br>
AsyncWorks <a style="font-style:italic"> <span style="color:red; font-weight:bold"> &nbsp;Svg &hearts;</span> &nbsp;(Söz verdiğimiz gibi)</a>
</body>
</html>
