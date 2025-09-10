<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>JFCM Taytay File Manager</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
<style>
  body { 
    font-family: 'Roboto', sans-serif; 
    display: flex; justify-content: center; align-items: center; 
    min-height: 100vh; 
    margin: 0; 
    background: linear-gradient(to bottom, rgba(0,0,0,0.2), rgba(0,0,0,0.7)),
                url('https://scontent.fmnl17-4.fna.fbcdn.net/v/t39.30808-6/476758085_1025079269656180_1508840151996929921_n.jpg?_nc_cat=105&ccb=1-7&_nc_sid=cc71e4&_nc_ohc=_ReU3GkJs5AQ7kNvwFJtgKv&_nc_oc=AdlPpscxYoBe9g4Dae6SXd8SP9S1_ioA207PYbYorCAA2QByrjCRxwWvleUdvUgoz7A&_nc_zt=23&_nc_ht=scontent.fmnl17-4.fna&_nc_gid=IJs6CRB_3-4tiip30yaB-Q&oh=00_AfUaAJAsEM1bu2MGoqc3nHFzGJXwRrAhdjpIYEt8dadGmQ&oe=68BB3577') 
                no-repeat center center/cover;
  }

  .container { 
    background: rgba(255,255,255,0.97); 
    padding: 40px; 
    border-radius: 25px; 
    width: 500px; 
    max-width: 95%; 
    text-align: center; 
    box-shadow: 0 15px 40px rgba(0,0,0,0.3); 
    transition: all 0.3s ease;
    animation: fadeIn 1s ease-in-out;
  }

  input, button { 
    width: 95%; 
    padding: 14px; 
    margin: 12px 0; 
    border-radius: 15px; 
    font-size: 1rem; 
    border: 1px solid #b2ebf2; 
    outline: none;
  }

  input:focus { border-color: #26a69a; box-shadow: 0 0 5px rgba(38,166,154,0.5); }

button { 
  background: #006400; /* dark green */
  color: #006400; 
  border: none; 
  cursor: pointer; 
  font-weight: 500;
  transition: 0.3s;
}

button:hover { 
  background: #004d00; /* darker green on hover */
}


  .error { color: red; margin-top: 10px; }

  /* Folder styles */
  .folder { 
    margin-top: 15px; 
    text-align: left; 
    border-radius: 12px; 
    overflow: hidden;
    box-shadow: 0 2px 6px rgba(0,0,0,0.1);
    background: #f1f8f9;
    transition: 0.3s;
  }

  .folder:hover { box-shadow: 0 4px 12px rgba(0,0,0,0.2); }

  .folder-header { 
    display: flex; align-items: center; 
    padding: 10px 15px; 
    cursor: pointer; 
    background: #26a69a; 
    color: white; 
    font-weight: 500;
    justify-content: space-between;
  }

  .folder-header span { display: flex; align-items: center; gap: 10px; }

  .folder-header:hover { background: #00796b; }

  .folder-content { 
    display: none; 
    padding-left: 20px; 
    padding-top: 10px; 
    padding-bottom: 10px; 
    background: #e0f2f1;
  }

  .folder-content a { 
    display: block; 
    text-decoration: none; 
    color: #00796b; 
    margin: 5px 0; 
    padding: 5px 10px; 
    border-radius: 8px;
    transition: 0.2s;
  }

  .folder-content a:hover { 
    background: #b2dfdb; 
    color: #004d40; 
  }

#login-box h2 {
  color: #006400; /* dark green */
  font-weight: 700;
}


  @keyframes fadeIn { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }

</style>
</head>
<body>
<div class="container" id="main-container">

  <!-- Login Box -->
  <div id="login-box">
    <h2>Jesus First Christian Ministries - Taytay</h2>
    <input type="email" id="email" placeholder="Enter Email">
    <input type="password" id="password" placeholder="Enter Password">
    <button onclick="siteLogin()">Login</button>
    <p class="error" id="error-message"></p>
  </div>

  <!-- Drive Box -->
  <div id="drive-box" style="display:none;">
    <h2>Folders</h2>
    <div id="foldersContainer"></div>
  </div>

</div>

<script type="module">
import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm';

const supabaseUrl = 'https://ustqtzorwudvspqhqsda.supabase.co';
const supabaseAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVzdHF0em9yd3VkdnNwcWhxc2RhIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTc0Mjc1NjUsImV4cCI6MjA3MzAwMzU2NX0.6xDg0MoJDhS_Gtg1TZ-xZ47Ws5nWnqUHEnqTfiAvkVA'; // replace with your anon key
const supabase = createClient(supabaseUrl, supabaseAnonKey);
const bucketName = 'private';
const foldersToShow = ['Preaching', 'Simbahay test', 'Sow', 'Google Drive'];

async function siteLogin() {
  const email = document.getElementById('email').value;
  const password = document.getElementById('password').value;

  const { data, error } = await supabase.auth.signInWithPassword({ email, password });
  if (error) {
    document.getElementById('error-message').textContent = 'Login failed: ' + error.message;
    return;
  }

  document.getElementById('login-box').style.display = 'none';
  document.getElementById('drive-box').style.display = 'block';
  listFolders();
}

function listFolders() {
  const container = document.getElementById('foldersContainer');
  container.innerHTML = '';
  foldersToShow.forEach(folderName => {
    createFolderElement(folderName, container);
  });
}

function createFolderElement(folderPath, parentContainer) {
  const folderDiv = document.createElement('div');
  folderDiv.className = 'folder';

  const folderHeader = document.createElement('div');
  folderHeader.className = 'folder-header';
  folderHeader.innerHTML = `<span>📁 ${folderPath}</span> <span>▼</span>`;
  folderDiv.appendChild(folderHeader);

  const filesContainer = document.createElement('div');
  filesContainer.className = 'folder-content';
  folderDiv.appendChild(filesContainer);

  folderHeader.onclick = () => {
    const isVisible = filesContainer.style.display === 'block';
    filesContainer.style.display = isVisible ? 'none' : 'block';
    folderHeader.querySelector('span:last-child').textContent = isVisible ? '▼' : '▲';

    if (!filesContainer.hasChildNodes() && !isVisible) {
      if(folderPath === 'Google Drive'){
        // Google Drive link
        const link = document.createElement('a');
        link.href = 'https://drive.google.com/drive/folders/1C43FvXt3pck1fbBit1jgOoIMYU8eQzCz?usp=sharing'; // Replace with your folder ID
        link.target = '_blank';
        link.textContent = 'Open Google Drive';
        filesContainer.appendChild(link);
      } else {
        loadFiles(folderPath, filesContainer);
      }
    }
  };

  parentContainer.appendChild(folderDiv);
}

async function loadFiles(folderPath, container) {
  container.innerHTML = 'Loading...';
  const { data: items, error } = await supabase.storage.from(bucketName).list(folderPath);
  container.innerHTML = '';

  if (error) {
    container.innerHTML = `<p style="color:red">${error.message}</p>`;
    return;
  }

  if (!items.length) {
    container.innerHTML = `<p>No files in this folder.</p>`;
    return;
  }

  for (const item of items) {
    if (item.type === 'folder') {
      const subfolderDiv = document.createElement('div');
      subfolderDiv.style.marginLeft = '20px';

      const subfolderHeader = document.createElement('div');
      subfolderHeader.className = 'folder-header';
      subfolderHeader.innerHTML = `<span>📁 ${item.name}</span> <span>▼</span>`;
      subfolderDiv.appendChild(subfolderHeader);

      const subfolderFilesContainer = document.createElement('div');
      subfolderFilesContainer.className = 'folder-content';
      subfolderDiv.appendChild(subfolderFilesContainer);

      subfolderHeader.onclick = () => {
        const isVisible = subfolderFilesContainer.style.display === 'block';
        subfolderFilesContainer.style.display = isVisible ? 'none' : 'block';
        subfolderHeader.querySelector('span:last-child').textContent = isVisible ? '▼' : '▲';
        if (!subfolderFilesContainer.hasChildNodes() && !isVisible) {
          loadFiles(`${folderPath}/${item.name}`, subfolderFilesContainer);
        }
      };

      container.appendChild(subfolderDiv);

    } else {
      const { data: urlData, error: urlError } = await supabase.storage
        .from(bucketName)
        .createSignedUrl(`${folderPath}/${item.name}`, 300);

      if (!urlError) {
        const link = document.createElement('a');
        link.href = urlData.signedUrl;
        link.target = '_blank';
        link.textContent = `📄 ${item.name}`;
        container.appendChild(link);
      }
    }
  }
}

window.siteLogin = siteLogin;
</script>
</body>
</html>
