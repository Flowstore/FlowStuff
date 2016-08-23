# FlowStuff
Apps Script testing

**************************Tech Help************************
//////Code.gs///////////

function doPost(request) {
  var sheets = SpreadsheetApp.openById('XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
  var params = request.parameters;

  var nR = getNextRow(sheets) + 1;

  if (params.token == "XXXXXXXXXXXXXXXXXX") {

    // PROCESS TEXT FROM MESSAGE
    var textRaw = String(params.text).replace(/^\s*tech_help\s*:*\s*/gi,'');
    var text = textRaw.split(/\s*;\s*/g);

    // FALL BACK TO DEFAULT TEXT IF NO UPDATE PROVIDED
    var category   = text[0] || "No Category Specified";
    var issue   = text[1] || "No Issue Specified";
    var what = text[2] || "No update provided";
    var howLong     = text[3] || "No update provided";

    // RECORD TIMESTAMP AND USER NAME IN SPREADSHEET
    sheets.getRangeByName('timestamp').getCell(nR,1).setValue(new Date());
    sheets.getRangeByName('user').getCell(nR,1).setValue(params.user_name);

    // RECORD UPDATE INFORMATION INTO SPREADSHEET
    sheets.getRangeByName('category').getCell(nR,1).setValue(category);
    sheets.getRangeByName('issue').getCell(nR,1).setValue(issue);
    sheets.getRangeByName('what').getCell(nR,1).setValue(what);
    sheets.getRangeByName('howLong').getCell(nR,1).setValue(howLong);
    sheets.getRangeByName('logged').getCell(nR,1).setValue("Received");
    
    var ticket = sheets.getRangeByName("ticket").getCell(nR, 1).getValue();

    var channel = "tech_help";

    postResponse(channel,params.channel_name,ticket,category,issue,params.user_name,what,howLong);


  } else {
    return;
  }
}

function getNextRow(sheets) {
  var timestamps = sheets.getRangeByName("timestamp").getValues();
  for (i in timestamps) {
    if(timestamps[i][0] == "") {
      return Number(i);
      break;
    }
  }
}


//////PostResponse.gs///////////
function postResponse(channel, srcChannel, ticket, category, issue, userName, what, howLong) {

  var payload = {
    "channel": "#" + channel,
    "username": "New Tech Issue",
    "icon_emoji": ":space_invader:",
    "link_names": 1,
    "attachments":[
       {
          "fallback": "This is an update from a Slackbot integrated into your organization. Your client chose not to show the attachment.",
          "pretext": " There is an update for Tech Support - *" + issue + "*. (Posted by @" + userName + " in #" + srcChannel + ")",
          "mrkdwn_in": ["pretext"],
          "color": "#D00000",
          "fields":[
             {
                "title":"Ticket Number",
                "value": ticket,
                "short":false
             },
             {
                "title":"Category",
                "value": category,
                "short":false
             },
             {
                "title":"Issue",
                "value": issue,
                "short":false
             },
             {
                "title":"What's happenning?",
                "value": what,
                "short":false
             },
             {
                "title":"How long have you had the issue?",
                "value": howLong,
                "short": false
             }
          ]
       }
    ]
  };

  var url = 'https://hooks.slack.com/services/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';
  var options = {
    'method': 'post',
    'payload': JSON.stringify(payload)
  };

  var response = UrlFetchApp.fetch(url,options);
}


**************************Tech Help 2************************
//////Code.gs///////////

function onOpen() {
  SpreadsheetApp.getUi() // Or DocumentApp or FormApp.
      .createMenu('Script Menu')
      .addItem('Update Slack', 'doPost')
      .addToUi();
}

function doPost(request) {
  var sheets = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = SpreadsheetApp.getActiveSheet();
  //var params = request.parameters;
  var cellRow = sheet.getActiveCell().getRow();
  var cellCol = sheet.getActiveCell().getColumn();
  
  if (cellCol == 9) {
  var ui = SpreadsheetApp.getUi(); // Same variations.
  var result = ui.alert(
     'Please confirm',
     'Do you want to update Slack?',
      ui.ButtonSet.YES_NO);

  // Process the user's response.
  if (result == ui.Button.YES) {
    // User clicked "Yes".
    ui.alert('Updating Slack');
    var user = sheets.getRangeByName("user").getCell(cellRow, 1).getValue();
    var ticket = sheets.getRangeByName("ticket").getCell(cellRow, 1).getValue();
    var status = sheets.getRangeByName("status").getCell(cellRow, 1).getValue();
    var delegated = sheets.getRangeByName("delegated").getCell(cellRow, 1).getValue();
    var update = sheets.getRangeByName("update").getCell(cellRow, 1).getValue();
    var eta = sheets.getRangeByName("eta").getCell(cellRow, 1).getValue();
    var channel = "tech_help";

    postUpdate(channel,user,ticket,status,delegated,update,eta); //params.channel_name,params.user_name
    
  } else {
    // User clicked "No" or X in the title bar.
    ui.alert('Come back when you are ready');
  }
  }
}

function onEdit(){
  var sheets = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = SpreadsheetApp.getActiveSheet();
  var cellRow = sheets.getActiveCell().getRow();
  var cellCol = sheets.getActiveCell().getColumn();
  var cell = sheet.getRange(cellRow, cellCol);
  
  if (cellCol == 9) {
    var ui = SpreadsheetApp.getUi().alert('Do not forget to update Slack!');
//    var response = ui.prompt('Slack Update', 'Do you want to update Slack?', ui.ButtonSet.YES_NO);
//
// // Process the user's response.
//    if (response.getSelectedButton() == ui.Button.YES) {
//      doPost(sheets,cellRow,cellCol);
//    } else if (response.getSelectedButton() == ui.Button.NO) {
//      ui.alert('Come back when you are ready');;
//    } else {
//      Logger.log('The user clicked the close button in the dialog\'s title bar.');
//    }
  }   
}


//////postUpdate.gs///////////

function postUpdate(channel, user, ticket, status, delegated, update, eta) { //srcChannel, userName, 

  var payload = {
    "channel": "@" + user,
    "username": "Tech BOT update",
    "icon_emoji": ":space_invader:",
    "link_names": 1,
    "attachments":[
       {
          "fallback": "This is an update from a Slackbot integrated into your organization. Your client chose not to show the attachment.",
          "pretext": " There is an update from Tech Support, regarding ticket number - *" + ticket + "*. (Posted by @" + user + ")",
          "mrkdwn_in": ["pretext"],
          "color": "#D00000",
          "fields":[
             {
                "title":"Ticket Number",
                "value": ticket,
                "short":false
             },
             {
                "title":"Status",
                "value": status,
                "short":false
             },
             {
                "title":"Has been delegated to",
                "value": delegated,
                "short":false
             },
             {
                "title":"Update notes",
                "value": update,
                "short":false
             },
             {
                "title":"Rough ETA for fix",
                "value": eta,
                "short": false
             }
          ]
       }
    ]
  };

  var url = 'https://hooks.slack.com/services/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';
  var options = {
    'method': 'post',
    'payload': JSON.stringify(payload)
  };

  var response = UrlFetchApp.fetch(url,options);
}
