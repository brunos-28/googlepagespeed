// Created by PageDart (https://pagedart.com)
// Adapted from Rick Viscomi https://dev.to/chromiumdev/a-step-by-step-guide-to-monitoring-the-competition-with-the-chrome-ux-report-4k1o
// Who Adapted it from https://ithoughthecamewithyou.com/post/automate-google-pagespeed-insights-with-apps-script by Robert Ellison

var scriptProperties = PropertiesService.getScriptProperties();
var pageSpeedApiKey = scriptProperties.getProperty('PSI_API_KEY');
var pageSpeedMonitorUrls = [
  'https://www.junglegym.co.uk',
  'https://www.junglegym.co.uk/sale',
  'https://www.junglegym.co.uk/climbing-frames',
  'https://www.junglegym.co.uk/swings',
  'https://www.junglegym.co.uk/swings/wooden-garden-play-equipment-totem-2-climb-1'
];

function monitor() {
  for (var i = 0; i < pageSpeedMonitorUrls.length; i++) {
    var url = pageSpeedMonitorUrls[i];
    var mobile = callPageSpeed(url);
    addRow(url, mobile);
  }
}

function callPageSpeed(url) {
  var pageSpeedUrl = 'https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=' + url + '&key=' + pageSpeedApiKey;
  options = {muteHttpExceptions: true};
  var response = UrlFetchApp.fetch(pageSpeedUrl, options);
  Logger.log(response.getContentText()); 
  var json = response.getContentText();
  return JSON.parse(json);
}

function addRow(url, mobile) {
  var spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = spreadsheet.getSheetByName('Sheet1');
  sheet.appendRow([
    Utilities.formatDate(new Date(), 'GMT', 'yyyy-MM-dd'),
    url,
    getPerformance(mobile)
  ]);
}

function getPerformance(data) {
  return data.lighthouseResult.categories.performance.score;
}
