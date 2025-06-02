<!DOCTYPE html>
<html>
<head>
  <title>Gmail API JavaScript Quickstart</title>
  <script src="https://apis.google.com/js/api.js"></script>
</head>
<body>
  <button id="authorize_button">Authorize</button>
  <button id="signout_button" style="display: none;">Sign Out</button>
  <pre id="content"></pre>

  <script>
    const CLIENT_ID = 'YOUR_CLIENT_ID';
    const API_KEY = 'YOUR_API_KEY';
    const SCOPES = 'https://www.googleapis.com/auth/gmail.readonly';
    let tokenClient;
    let gapiInited = false;
    let gisInited = false;

    function gapiLoaded() {
      gapi.load('client', initializeGapiClient);
    }

    function gisLoaded() {
      tokenClient = google.accounts.oauth2.initTokenClient({
        client_id: CLIENT_ID,
        scope: SCOPES,
        callback: '', // defined at requestAccessToken time
      });
      gisInited = true;
      maybeEnableButtons();
    }

    function initializeGapiClient() {
      gapi.client.init({
        apiKey: API_KEY,
      }).then(function() {
        gapiInited = true;
        maybeEnableButtons();
      });
    }

    function maybeEnableButtons() {
      if (gapiInited && gisInited) {
        document.getElementById('authorize_button').style.visibility = 'visible';
      }
    }

    function handleAuthClick(event) {
      tokenClient.requestAccessToken({prompt: 'consent'});
    }

    function handleSignoutClick(event) {
      const token = google.accounts.oauth2.getAuthInstance().getToken();
      if (token) {
        google.accounts.oauth2.revoke(token.access_token);
      }
      document.getElementById('authorize_button').style.visibility = 'visible';
      document.getElementById('signout_button').style.visibility = 'hidden';
      document.getElementById('content').innerText = '';
    }

    function listEmails() {
      gapi.client.gmail.users.messages.list({
        'userId': 'me',
        'q': 'is:unread'
      }).then(function(response) {
        const messages = response.result.messages;
        if (messages && messages.length > 0) {
          const messageId = messages[0].id;
          getMessage(messageId);
        } else {
          document.getElementById('content').innerText = 'No new messages.';
        }
      });
    }

    function getMessage(messageId) {
      gapi.client.gmail.users.messages.get({
        'userId': 'me',
        'id': messageId
      }).then(function(response) {
        const headers = response.result.payload.headers;
        let subject = '';
        for (let i = 0; i < headers.length; i++) {
          if (headers[i].name === 'Subject') {
            subject = headers[i].value;
            break;
          }
        }
        document.getElementById('content').innerText = 'Subject: ' + subject;
      });
    }

    document.getElementById('authorize_button').onclick = handleAuthClick;
    document.getElementById('signout_button').onclick = handleSignoutClick;
  </script>
</body>
</html>
