## Deploying REST server using IBM Code Engine
1. Log in and set the environment
```
ibmcloud login --sso
```
set resource group (in our case it's "default")
```
ibmcloud target -g default 
```
(optionally, if not created) create project named "WA-workshop" in IBM Code Engine service
```
ibmcloud ce project create --name WA-workshop --endpoint public
```
(optionally, if not selected) select project named "WA-workshop" as the active one
```
ibmcloud ce project select --name WA-workshop
```
from now, all applications will be deployed in project "WA-workshop" namespace

2. Deploy the app

IMPORTANT! execute following command being inside the directory: webfrontend-widget

IMPORTANT! remove previous src/node_modules directory and src/package-lock.json file - it will be recreated in Code Engine
```
ibmcloud ce app create --name webfrontend-widget --build-source ./src --strategy buildpacks --min 1 --max 1
```
3. Record the web page URL
```
ibmcloud ce app list
```


## Deploying REST server using IBM CloudFoundry

1. Log in and set the environment
```
ibmcloud cf login --sso
ibmcloud target -g default 
ibmcloud target --cf
```
2. Run applications in CloudFoundry
```
ibmcloud cf push -f manifest.yml
```
3. Record the web page URL
```
ibmcloud cf apps
```
4. Check the logs
```
ibmcloud cf logs webhook-restapi-server --recent
```

## How to customize web page to include the Watson Assistant web widget
1. Copy paste JS code snippet from Watson Assistant
2. Edit file `src/src/index.html`
add `<script>` section below comment text: `<!-- Place your JS code snippet from Watson Assistant Web integration-->`
3. Update application

a. Option 1 - for IBM Code Engine
```
ibmcloud ce app update --name webfrontend-widget --build-source ./src --strategy buildpacks --min 1 --max 1
```

b. Option 2 - for IBM Cloud Foundry
```
ibmcloud cf push -f manifest.yml
```

## Customizing Widget Icon
As the "icing on the cake", I present how to customize the web widget so that it has its own icon and information in the editbox for interaction with the chatbot

1. Add the custom button to your 'HTML' file, with your svg icon
```
  <button type="button" class="chatLauncher" style="display:none;">
    <svg xmlns="http://www.w3.org/2000/svg" 
         xmlns:xlink="http://www.w3.org/1999/xlink" 
         version="1.1" 
         x="0px" y="0px" 
         viewBox="0 0 32 32" 
         xml:space="preserve">
      <defs/>
      <g>
        <image x="0" y="0" width="100%" height="100%" 
        xlink:href="https://app.diagrams.net/img/lib/ibm/users/integrated_digital_experiences.svg"/>
      </g>
    </svg>
  </button>
```

2. style it with css e.g. right at the end of `header`
```
  <style>
    button.chatLauncher {
      position: fixed;
      bottom: var(--WatsonAssistantChat-LAUNCHER-position-bottom);
      right: var(--WatsonAssistantChat-LAUNCHER-position-bottom);
      z-index: var(--WatsonAssistantChat-BASE-z-index);
      border: var(--WatsonAssistantChat-BASE-border-width-med) solid transparent;
      border-radius: var(--WatsonAssistantChat-BASE-border-radius-xsmall);
      margin: 0;
      text-decoration: none;
      box-shadow: var(--WatsonAssistantChat-BASE-box-shadow);
      cursor: pointer;
      text-align: left;
      -webkit-appearance: none;
      -moz-appearance: none;
      width: 64px;
      height: 64px;
      padding: 0;
      opacity: 1;
      animation-duration: 0.5s;
      animation: WACLauncherIn var(--WatsonAssistantChat-CARBON-duration--moderate-01) var(--WatsonAssistantChat-CARBON-easing-entrance-expressive) both;
      transition-duration: 0.5s;
      transition: background .25s ease-in-out,transform .15s ease;    
    
    }
    button.chatLauncher:enabled {
      background: #FFFFFF;
    }
    
    button.chatLauncher:enabled:hover {
      background: #BD9842;
    }
    button.chatLauncher:enabled:active {
      box-shadow: inset 0 0 0 var(--WatsonAssistantChat-BASE-border-width-small) var(--WatsonAssistantChat-CARBON-ui-02);
      opacity: .8;
    }
    button.chatLauncher:focus {
      box-shadow: inset 0 0 0 var(--WatsonAssistantChat-BASE-border-width-small) var(--WatsonAssistantChat-CARBON-ui-02);
    }
    #WACContainer.WACContainer .WAC__inputContainer .WAC__TextArea .WAC__TextArea-sizer, #WACContainer.WACContainer .WAC__inputContainer .WAC__TextArea .WAC__TextArea-textarea {
    padding: 0.25rem 1rem !important;
    }
  </style>
```

3. change the default 'JS' code snipped to customize messages right at the end of the `body`
```
  <script>
    window.watsonAssistantChatOptions = {
        integrationID: "YOUR_INTEGRATION_ID", // The ID of this integration.
        region: "eu-de", // The region your integration is hosted in.
        serviceInstanceID: "YOUR_SERVICE_INSTNCE_ID", // The ID of your service instance.
        showLauncher: false, // disable standard launcher
        onLoad: function(instance) { 
          const customLanguagePack = {
            "input_placeholder": "Napisz swoje pytanie",
            "homeScreen_conversationStarterLabel": "Dzień dobry, o czym porozmawiamy",
		        "errors_communicating": "Spróbuj ponownie za jakiś czas"
          };
	  
          instance.updateLanguagePack(customLanguagePack);
	  	  
          const button = document.querySelector('.chatLauncher');
  
          // Add the event listener to open your web chat.
          button.addEventListener('click', function clickListener() {
            instance.openWindow();
          });
  
          // Render the web chat. Nothing appears on the page, because the launcher is
          // hidden and the web chat window is closed by default.
          instance.render().then(function() {
            // Now that web chat has been rendered (but is still closed), we make the
            // custom launcher button visible.
            button.style.display = 'block';
            button.classList.add('open');
          });
        }
    };

    setTimeout(function(){
      const t=document.createElement('script');
      t.src="https://web-chat.global.assistant.watson.appdomain.cloud/versions/" + (window.watsonAssistantChatOptions.clientVersion || 'latest') + "/WatsonAssistantChatEntry.js"
      document.head.appendChild(t);
    });
  </script>
```