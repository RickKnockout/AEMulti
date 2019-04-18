const rp = require('request-promise'),
	cheerio = require('cheerio'),
	inq = require('inquirer'),
	requestpromise = require('request')
var request = rp.defaults({
		jar: true
	}),
	baseStructuresTable = null,
	baseStructuresTrs = null,
	structuresObj = [],
	baseQueueTable = null,
	baseQueueForm = null,
	baseQueueTrs = [],
	params = null,
	start = null,
	end = null,
	baseQueueLength = null,
	queueInfo = null,
	canQueueInfo = null,
	maxStructures = {},
	baseCounter = [],
	canRun = true,
	firstRun = true,
  proxyUser,
  proxyPass,
	d = [],
	email, password, server, verbose = false,
	proxy = "",
  maxMR = "20",
	Agent = require('socks5-https-client/lib/Agent');
  var intervalMins = 5 // replaced with user input.
  var intervalMs = intervalMins * 60 * 1000

var questions = [{
	type: 'input',
	name: 'email',
	message: 'Login Email:',
  validate: function(value){
    var pass = value.match(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/g);
    if (pass) return true;
    return "Please enter a valid email."
  }
}, {
	type: 'password',
	name: 'password',
	message: 'Login Password:'
}, {
	type: 'list',
	name: 'server',
	message: 'Select Server:',
	choices: ['alpha', 'beta', 'ceti', 'delta', 'epsilon', 'fenix', 'gamma', 'helion','ixion','juno','kappa','lyra','mira','nova','omega','pegasus','quantum','rigel','sigma','typhon','utopia', 'lynx', 'mystic' ]
},{
  type: 'input',
  name: 'sleepCycle',
  message: 'How often to repeat loop (in minutes)? Note: this answer is randomized to +/- 10% every loop.',
  validate: function(value){
    var pass = value.match(/([0-9])/g);
    if (pass) return true;
    return "Please enter a valid number."
  }
},{
	type: 'confirm',
	name: 'verbose',
	message: 'Verbose Output?',
	default: true
}, {
	type: 'confirm',
	name: 'proxyToggle',
	message: 'Use Proxy?',
	default: true
},{
  type: 'input',
  name: 'maxMR',
  message: 'Max Metal Refineries Level?',
  validate: function(value){
    var pass = value.match(/([0-9])/g);
    if (pass) return true;
    return "Please enter a valid number."
  }
},{
  type: 'confirm',
  name: 'fleetBuilderToggle',
  message: 'Queue Fleet?',
  default: false
}];

var proxyQuestions = [{
	type: 'input',
	name: 'proxy',
	message: 'HTTP proxy IP: (e.g. 127.0.0.1)'
}, {
	type: 'input',
	name: 'port',
	message: 'HTTP Port: '
}, {
  type: 'input',
  name: 'proxyUser',
  message: 'Proxy username: '
},{
  type: 'password',
  name: 'proxyPass',
  message: 'Proxy password: '
}]

// Prepare questions for fleet building.. we will ask for quantities of each next.
var fleetQuestions = [{
  type: 'checkbox',
  name: 'fleetTypes',
  message: 'Select which units to queue',
  choices: ['Fighters','Bombers','Corvette', 'Recycler', 'Destroyer', 'Frigate', 'Ion Frigate', 'Cruiser', 'Carrier', 'Heavy Cruiser', 'Battleship', 'Fleet Carrier', 'Dreadnought', 'Titan', 'Leviathan', 'Death Star']
},{
  type: 'confirm',
  name: 'fillHangars',
  message: 'Queue Fighters to Fill Hangars?',
  default: false
}]

// We will push data to this array later based on the user input from above.
var fleetQuantQuestions = []
var proxyIP;
var proxyPort
async function prepareBaseLocs() {
	if (firstRun) {
		inq.prompt(questions) // first main questions
			.then(answers => {
				email = answers.email;
				password = answers.password;
				server = answers.server;
				verbose = answers.verbose;
        maxMR = answers.maxMR;
        fleetBuilder = answers.fleetBuilderToggle;
        intervalMins = answers.sleepCycle;

				if (answers.proxyToggle == true) { // if we're using a proxy...
					inq.prompt(proxyQuestions) // http proxy questions
						.then(answers => {
							proxyIP = answers.proxy
              proxyPort = answers.port
              proxyUser = answers.proxyUser
              proxyPass = answers.proxyPass
						}).then(() => {
							if (proxyIP) {
								request = rp.defaults({
									'jar': true,
                  proxy: 'http://' + proxyUser + ":" + proxyPass + "@" + proxyIP + ':' + proxyPort
									//agentClass: Agent,
									//agentOptions: {
										//socksHost: proxy, // Defaults to 'localhost'.
										//socksPort: proxyPort // Defaults to 1080.
									//}
								});
                Log('Using proxy: http://' + proxyIP + ':' + proxyPort)
							}
						}).then(() => {
							testProxy();
						})
				} else {
					testProxy();
				}
			})
	};
}

var cred = ""

// Login and pull base locations here.
async function getBaseLocs() {
	return new Promise(async resolve => {
		console.log('> Navigating to ' + server + '.astroempires.com...')
		console.log('> Logging in as ' + email + '...')
		var options = {
			simple: false,
			url: "https://" + server + ".astroempires.com/login.aspx",
			form: {
				"server": "https://" + server + ".astroempires.com/login.aspx",
				"email": email,
				"pass": password,
				"post_back": true,
				"navigator": "Netscape",
				"hostname": "www.astroempires.com",
				"javascript": true
			}
		}
		request.post(options)
			.then(function(body) {
				//Log('Succeeded with response %d ' + response.statusCode)
				console.log('> Grabbing info...')
				request.get("https://" + server + ".astroempires.com/empire.aspx?view=bases_events")
					.then(function(body) {
						$ = cheerio.load(body)
						var c = $('#empire_events > tbody > tr:nth-child(2) > td > table > tbody')
							.children()
							.length
            cred = $('#credits').next().text()
						for (let i = 0, f = 2; f < c + 1; i++, f++) {
							let e = $('#empire_events > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(' + f + ') > td:nth-child(1) > a')
								.attr('href')
							d.push(e) //d[0] = base.aspx?base=12345
							baseCounter[i] = {
								"ID": e,
								"Build": "True"
							}
						}
						if (d.length < 1) {
							console.log('> Error logging in!')
							return 0;
						} else {
							console.log('> Found total of (' + d.length + ') bases.')
              console.log('')
							resolve(d);
						}
					})
					.catch(function(err) {
						console.error(err)
					})
			})
			.catch(function(err) {
				console.error(err)
			})
	})
}

var baseRuns;
var flagCounter = 0
var fleetUnits = [] // Array to store how many fleets to queue
// Function to only output if verbose is true.
function Log(x, y) {
	if (verbose == true) {
		if (y) {
			console.log(x, y);
		} else {
			console.log(x);
		}
	}
}

// Randomize sleep values.. multiply by 0.9-1.1 (+/- 10% sleep time)
// Uses Math.floor to return integer since we use milliseconds anyway

function randomSleep(time){
 return Math.floor(time * getRandomFloat(0.9, 1.1))
}

// Used for randomizing sleep
function getRandomFloat(min, max){
  return Math.random() * (max - min) + min;
}

function testProxy() {
	console.log('> Finding out our IP...')
	request.get("https://api.ipify.org?format=json")
		.then(function(body) {
			$ = cheerio.load(body)
			console.log('Our IP is: ' + body)
		})
		.then(function() {
			console.log('> Proceeding with logic loop...')
			prepAutoBuild();
		})
}

// Ask questions for which bases to run the script on
async function prepAutoBuild() {
  var j = await getBaseLocs();
	var baseQuestions = [{
		type: 'checkbox',
		name: 'abBases',
		message: 'Select Bases to Build On:',
		choices: []
	}]
	for (i = 0; i < j.length; i++) {
		baseQuestions[0].choices.push(j[i]);
	}
  if (fleetBuilder){
   await fleetPrep(); // Jump to the questions for fleet prep
  }
	inq.prompt(baseQuestions) // Which bases to queue on?
  .then(async answers => {
    console.log('[*] Current Credits: ' + cred)
    d = answers.abBases;
		for (i = 0; i < answers.abBases.length; i++) {
			let base = answers.abBases[i];
			await autoBuild(base)
		}
	})
  .then(async () => {
    console.log("")
    intervalMs = randomSleep(intervalMs)
    console.log('> Waiting ' + intervalMs + 'ms (' + intervalMs / 60000 + ' minutes) before next cycle...')
		baseRuns = setInterval(async () => {
			for (i = 1; i < d.length; i++) {
				let base = d[i];
				await autoBuild(base)
			}
		}, intervalMs)
	})
}

async function fleetPrep(){
  inq.prompt(fleetQuestions)
  .then(async answers =>{
    console.log(answers)
    for (i = 0; i < answers.fleetTypes.length; i++){
     let tempQuestion = {
       type: 'input',
       name: answers.fleetTypes[i],
       message: 'How many ' + answers.fleetTypes[i] + ' to keep in queue?',
       validate: function(value){
         var pass = value.match(/([0-9])/g);
         if (pass) return true;
         return "Please enter a valid number."
       }
     }
     fleetQuantQuestions.push(tempQuestion)
  }
  })
  .then(() => {
    inq.prompt(fleetQuantQuestions)
    .then(async answers => {
    for (i=0; i < answers.length; i++){
      let tempUnit = {
        unit: fleetQuantQuestions[i].name,
        amount: answers.fleetQuantQuestions[i]
      }
      console.log('teumpUnit: ' + tempUnit)
      console.log('fleetUnits: ' + fleetUnits)
      fleetUnits.push(tempUnit);
    }
    })
  })
  Promise.resolve();
}

// Function to loop through building after the 15 minute wait
async function logicLoop() {
    console.log('[*] Current Credits: ' + cred)
		let promise = new Promise (async (resolve) => {
      for (i = 0; i < d.length; i++) {
			let base = d[i];
			await autoBuild(base) }
      resolve();
    });
    await promise;
    console.log("")
    intervalMs = randomSleep(intervalMs)
    console.log('> Waiting ' + intervalMs + 'ms (' + intervalMs / 60000 + ' minutes) before next cycle...')
		baseRuns = setInterval(async function() {
			for (i = 1; i < d.length; i++) {
				let base = d[i];
				await autoBuild(base)
			}
		}, intervalMs)
	}

// Function to start autobuilding on each base.
async function autoBuild(base) {
	return new Promise(async resolve => {
		console.log("")
		for (let i = 0; i < baseCounter.length; i++) {
			if (baseCounter[i].ID == base && baseCounter[i].Build == "False") {
				flagCounter++
				console.log('Skipping base ' + base + ' -- Queues were full.')
				if (flagCounter >= d.length) {
					firstRun = false;
					sleepLonger();
				} else {
          let nap = randomSleep(3000);
          Log('    Waiting ' + nap + 'ms before going back..');
					await new Promise(resolve => setTimeout(resolve, nap));
					resolve()
          return;
				}
			}
		}
		if (flagCounter < d.length) {
			flagCounter = 0;
			request.get("https://" + server + ".astroempires.com/" + base + "&view=structures").then(async function(body) {
				$ = cheerio.load(body)
        let credTemp = $('#credits').next().text()
				baseStructuresTable = $('#base_structures > tbody > tr:nth-child(2) > td > table > tbody');
				baseStructuresTrs = baseStructuresTable.children('tr')
				baseQueueTable = $("#base-queue_content > form > table > tbody");
				params = $('#base-queue > tbody > tr:nth-child(2) > td > script')
					.html();
				let regex = /([0-9])\w+/
				console.log('[-] Base: ' + base)
        if (cred != credTemp){
          cred = credTemp
          console.log('[*] Current Credits: ' + cred)
        }
				structuresObj = getBaseStructures(baseStructuresTrs);
				start = params.search('base.aspx');
				end = params.search('add_stack');
				params = params.substring(start, end + 'add_stack'.length)
					.replace('method=ajax&', '');
				baseQueueLength = baseQueueTable.children()
					.length
				queueInfo = getQueueInfo(baseQueueLength)
				canQueueInfo = getCanQueueInfo(baseQueueLength)
				var construct = null;
				if (canQueueInfo.length > 0) {
					let queueInfo = {};
					queueInfo = getQueueInfo(baseQueueTrs);
					construct = calculateBuild(structuresObj, canQueueInfo, queueInfo, maxStructures);
				} else {
					if (construct == null) {
						console.log('Queues Full!')
						for (let i = 0; i < baseCounter.length; i++) {
							if (baseCounter[i].ID == base) {
								//Log('Flagging ' + baseCounter[i].ID + ' False');
								baseCounter[i].Build = "False";
                let nap = randomSleep(3000)
                Log('    Waiting ' + nap +'ms before going back..');
								await new Promise(resolve => setTimeout(resolve, nap));
								resolve();
                return;
							}
						}
					}
				}
				if (construct != null) {
					Log('    > Attempting to build: ' + construct)
					let addUrl = params;
					addUrl = addUrl + "&_q=" + (new Date)
						.getTime();
					options = {
						simple: false,
						url: "https://" + server + ".astroempires.com/" + addUrl,
						form: {
							"server": "https://" + server + ".astroempires.com/",
							"item": construct,
							"post_back": true,
							"navigator": "Netscape",
							"hostname": "www.astroempires.com",
							"javascript": true
						}
					}
					request.post(options).then(async function(body) {
            cred = $('#credits').next().text()
						console.log('    [+] Successfully queued: ' + construct)
            console.log('');
            let nap = randomSleep(3000)
						Log('    Waiting ' + nap +'ms before going back..');
						await new Promise(resolve => setTimeout(resolve, nap));
						resolve();
					}).catch(function(err) {
						console.error(err)
					})
				} else {
          if (construct == null){
              for (let i = 0; i < baseCounter.length; i++) {
							if (baseCounter[i].ID == base) {
								//Log('Flagging ' + baseCounter[i].ID + ' False');
								baseCounter[i].Build = "False";
                let nap = randomSleep(3000)
								Log('    Waiting ' + nap + 'ms before going back..');
								await new Promise(resolve => setTimeout(resolve, nap));
								resolve();
                return;
							}
						}
          }
        }
			})
		}
	})
}

function sleepLonger() {
	console.log('')
	console.log('********************')
	console.log('[*] All of our bases have full queues.')
  let longNap = randomSleep(900000)
	console.log('[*] Setting a ' + longNap / 60000 + ' minute wait before checking again.')
  console.log('********************')
	clearInterval(baseRuns);
	setTimeout(function() {
		flagCounter = 0;
		baseCounter = [];
		setTimeout(logicLoop, longNap) //Run getBaseLocs(prepAutoBuild) in 15 mins
	}, 5000)
}

function getCanQueueInfo() {
	var canQueueInfo = [];
	var optionInfos = $('#item > option')
	var log = "";
	for (var i = 0, k = 1; i < optionInfos.length; i++, k++) {
		let item = $('#item > option:nth-child(' + k + ')')
		log = log + item.text() + "\n"; // #item > option:nth-child(1)
		Log('CanQueue: ', item.text())
		canQueueInfo.push(item.text());
	}
	return canQueueInfo;
}

function getQueueInfo() {
	var queueInfo = {};
	if (baseQueueLength > 1) { // if length is one, it stores just what we CAN queue
		for (let j = 0, k = 1; j < baseQueueLength; j++, k++) {
			// logs current queue into an array
			var queueStructure = $('#base-queue_content > form > table > tbody > tr:nth-child(' + k + ') > td:nth-child(1)')
				.text();
			if (queueInfo[queueStructure] == "undefined" || queueInfo[queueStructure] == null) {
				queueInfo[queueStructure] = 1;
			} else {
				queueInfo[queueStructure]++;
			}
		}
	}
	return queueInfo;
}

function getBaseStructures(baseStructuresTrs) {
	structuresObj = [];
	var strucInfo = "";
	for (let i = 1; i < baseStructuresTrs.length; i++) {
		if (i % 2 == 0) {
			var baseStructuresTds = $('#base_structures > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(' + i + ')')
				.children('.dropcap')
			var baseStructureTd = baseStructuresTds[0];
			var baseStructureHtml = $(baseStructuresTds[0])
				.html();
			var structureName = $(baseStructureTd)
				.children('b')
				.children('a')
				.html();
			var urbanPopulation;
			if (structureName == "Urban Structures") {
				var pos = baseStructureHtml.search("bases fertility");
				urbanPopulation = baseStructureHtml.substring(pos + 17, pos + 18);
				//Log('fertility: ' + urbanPopulation)
			}
			var pos1 = baseStructureHtml.search("\\u0028Level ");
			var structureLevel;
			if (pos1 < 1) {
				structureLevel = 0;
			} else {
				structureLevel = baseStructureHtml.substring(pos1 + 7, pos1 + 9);
				if (structureLevel.search("\\u0029") > 0) {
					structureLevel = structureLevel.substring(0, structureLevel.search("\\u0029"));
				}
			}
			//Log('Level: ' + structureLevel)
			var production = 0;

			if (structureName == "Metal Refineries") {
				var subStructureHtml = $(baseStructureTd)
					.children('div')
					.html()
				var pos2 = subStructureHtml.search("\\u0029");
				if (pos2 > 0) {
					production = subStructureHtml.substring(pos2 - 1, pos2);
					//Log('Prod: ' + production)
				}
			}

			//#base_structures > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(2) > td:nth-child(3)
			var cost = parseInt($('#base_structures > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(' + i + ') > td:nth-child(3)')
				.html()
				.replace(",", ""));
			var structureEnergy = $('#base_structures > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(' + i + ') > td:nth-child(4)')
				.html()
			if (structureEnergy == null || structureEnergy == "") structureEnergy = 0;
			structureEnergy = 0 + parseInt(structureEnergy);
			//Log('Energy: ', structureEnergy)
			var structureStatusHtml = $('#base_structures > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(' + i + ') > td:nth-child(7)')
				.html()
			//out popul -- out area
			var structureStatus = "free";

			if (structureStatusHtml.search("Build") > -1) {
				structureStatus = "free";
			} else if (structureStatusHtml.search("working") > -1) {
				structureStatus = "waiting";
			} else if (structureStatusHtml.search("req") > -1) {
				structureStatus = "req upgrade";
			} else if (structureStatusHtml.search("out popul") > -1) {
				structureStatus = "out popul";
			} else if (structureStatusHtml.search("out area") > -1) {
				structureStatus = "out area";
			} else if (structureStatusHtml.search("out energy") > -1) {
				structureStatus = "out energy";
			} else if (structureStatusHtml.search("out cred") > -1) {
				structureStatus = "out credits";
			} else {
				structureStatus = "building";
				structureLevel++;
				cost = Math.ceil(cost * 1.5);
				//Log('New Cost: ' + cost)
			}
			//Log('Status: ', structureStatus)
			//Log(structureName + " lvl: " + structureLevel + " cost: " + cost + " status: " + structureStatus);
			var structureForm = $('#base_structures > tbody > tr:nth-child(2) > td > table > tbody > tr:nth-child(' + i + ') > td:nth-child(7) > form')
			if (structureName == "Robotic Factories") {
				production = 2;
			}
			if (structureName == "Shipyards") {
				production = 2;
			}
			if (structureName == "Nanite Factories") {
				production = 4;
			}
			if (structureName == "Android Factories") {
				production = 6;
			}
			if (structureName == "Orbital Shipyards") {
				production = 8;
			}

			var structureInfo = {};
			structureInfo.structureName = structureName;
			structureInfo.structureLevel = structureLevel;
			structureInfo.structureEnergy = structureEnergy;
			structureInfo.cost = cost;
			structureInfo.structurePopulation = -1;
			structureInfo.structureArea = -1;
			structureInfo.structureStatus = structureStatus;
			structureInfo.structureForm = structureForm;
			structureInfo.production = production;

			if (structureName == "Urban Structures") {
				structureInfo.structurePopulation = urbanPopulation;
			}
			if (structureName == "Terraform") {
				structureInfo.structurePopulation = 0;
				structureInfo.structureArea = 5;
			}
			if (structureName == "Multi-Level Platforms") {
				structureInfo.structurePopulation = 0;
				structureInfo.structureArea = 10;
			}
			if (structureName == "Orbital Base") {
				structureInfo.structurePopulation = 10;
			}
			if (structureName == "Orbital Shipyards") {
				structureInfo.structureArea = 0;
			}
			if (structureName == "Jump Gate") {
				structureInfo.structureArea = 0;
			}

			for (var aa in structureInfo) {
				strucInfo = strucInfo + aa + " " + structureInfo[aa] + "\n";
			}
			strucInfo = strucInfo + "\n";
			structuresObj.push(structureInfo);
		}
	}

	//Log(strucInfo); //show all the structures info

	return structuresObj;
}

function calculateBuild(structuresObj, canQueueInfo, queueInfo, maxStructures) {
	var isFree = ($("advertising")
		.length >= 1);
	var log = "";
	var mrLv = getLevelFromStructuresObj(structuresObj, "Metal Refineries", queueInfo);

	//looking for the best value/price ratio construct-structure
	var conStruct;
	var temp;
	//building type : 1.production 2.power 3.econ 4.population 5.area
	var type = 0;
	log = "";
	var syLv = getLevelFromStructuresObj(structuresObj, "Shipyards", queueInfo);
	var spLv = getLevelFromStructuresObj(structuresObj, "Spaceports", queueInfo);
	var robotLv = getLevelFromStructuresObj(structuresObj, "Robotic Factories", queueInfo);
	var econLv = getLevelFromStructuresObj(structuresObj, "Economic Centers", queueInfo);
	var NaniLv = getLevelFromStructuresObj(structuresObj, "Nanite Factories", queueInfo);
	var AndroidLv = getLevelFromStructuresObj(structuresObj, "Android Factories", queueInfo);
	var OSPLv = getLevelFromStructuresObj(structuresObj, "Orbital Shipyards", queueInfo);

	var maxLv = 1; // Max spaceports level
	if (mrLv >= 18) {
		maxLv = 20;
	} else if (mrLv >= 15) {
		maxLv = 15;
	} else if (mrLv >= 10) {
		maxLv = 10;
	} else if (mrLv >= 8) {
		maxLv = 5;
	}

	Log("MR lvl: " + mrLv + " SP Max Level: " + maxLv);

	if (mrLv >= 4 && spLv < maxLv && isExists(canQueueInfo, "Spaceports")) {
		return "Spaceports";
	}

	if ((mrLv - robotLv) > 4 && isExists(canQueueInfo, "Robotic Factories")) {
		//Log("MRLvl - RoboticsLvl > 4")
		return "Robotic Factories";
	}

	if (NaniLv > 0 && (econLv - NaniLv) < 1 && isExists(canQueueInfo, "Economic Centers") && econLv < 12) {
		//Log("NanitesLvl > 0, EcoLvl - NanitesLvl < 1")
		return "Economic Centers";
	}

	conStruct = [];
  // if we can queue terraform, add it to the array...
	if (isExists(canQueueInfo, "Terraform")) {
		conStruct.push("Terraform");
	}
  // if we can queue mlp, add that...
	if (isExists(canQueueInfo, "Multi-Level Platforms")) {
		conStruct.push("Multi-Level Platforms");
	}
	var mt = 0;
// if either one are in the array, calculate the cheapest price for area, store it.
	if (conStruct.length != 0) {
		var marginalArea = getCost(structuresObj, conStruct, 5, 0, 0);
		mt = marginalArea.cheapestPrice;
	}

  // New array for population calculations..
	conStruct = new Array("Urban Structures");
	if (isExists(canQueueInfo, "Orbital Base")) {
		conStruct.push("Orbital Base");
	}
  if (isExists(canQueueInfo, "Biosphere Modification")){ // Added new, untested!
    conStruct.push("Biosphere Modification");
  }

	var marginalPopulation = getCost(structuresObj, conStruct, 4, 0, mt);

	var solarEnergy, GasEnergy;
	for (bb in structuresObj) {
		if (structuresObj[bb].structureName == "Solar Plants") {
			solarEnergy = structuresObj[bb].structureEnergy;
		} else if (structuresObj[bb].structureName == "Gas Plants") {
			GasEnergy = structuresObj[bb].structureEnergy;
		}
	}
	var solarLv = getLevelFromStructuresObj(structuresObj, "Solar Plants", queueInfo);
	var gasLv = getLevelFromStructuresObj(structuresObj, "Gas Plants", queueInfo);
	conStruct = new Array();
	if (isExists(canQueueInfo, "Fusion Plants")) {
		conStruct.push("Fusion Plants");
	}
	if (isExists(canQueueInfo, "Antimatter Plants")) {
		conStruct.push("Antimatter Plants");
	}
  if (isExists(canQueueInfo, "Orbital Plants")){
    conStruct.push("Orbital Plants");
  }
	if (isExists(canQueueInfo, "Antimatter Plants") && GasEnergy <= 2 && solarEnergy <= 2) {} else {
		if ((GasEnergy <= 2 && solarEnergy <= 2 && solarLv < 3) || (solarEnergy == 3 && solarLv < 6) || solarEnergy > 3) {
			conStruct.push("Solar Plants");
		}
		if ((GasEnergy <= 2 && solarEnergy <= 2 && gasLv < 3) || (GasEnergy == 3 && gasLv < 6) || GasEnergy > 3) {
			conStruct.push("Gas Plants");
		}
	}
	//var mp = Math.max(500,marginalPopulation.cheapestPrice);
	var mp = marginalPopulation.cheapestPrice; // Calculate cheapest population, store that..
	var marginalEnergy = getCost(structuresObj, conStruct, 2, mp, mt); // Calculate cheapest energy based off of population and area cost, store that...
	conStruct = new Array("Metal Refineries", "Robotic Factories");
	if (!isFree || NaniLv < 5) {
		conStruct.push("Nanite Factories");
	}

	if (!isFree || AndroidLv < 5) {
		conStruct.push("Android Factories");
	}

	if (!isFree || OSPLv < 5) {
		conStruct.push("Orbital Shipyards");
	}

	if (syLv < maxStructures.Shipyards) {
		conStruct.push("Shipyards");
	}

	var marginalProduction = getCost(structuresObj, conStruct, 1, mp, mt, marginalEnergy.cheapestPrice); 
	var cheapestProduction = marginalProduction.cheapest; // Calculate cheapest production based off of energy, area, population costs.. store that
	var needEnergy = "";
	for (var bb in structuresObj) {
		if (structuresObj[bb].structureName == cheapestProduction) {
			needEnergy = structuresObj[bb].structureEnergy;
		}
	}

	maxLv = maxMR; // Max level set at beginning of queries.

	if (mrLv >= maxLv && cheapestProduction == "Metal Refineries") {
		console.log('Stopped building, reached max level');
		return null;
	}

  if (mrLv >= maxLv && mrLv - robotLv <= 5 && mrLv - NaniLv <= 10 && mrLv - AndroidLv <= 15){
    console.log('Stopped building, reached max level');
    return null;
  }

	if (isExists(canQueueInfo, cheapestProduction)) {
		Log("Returning Cheapest Production: " + cheapestProduction)
		return cheapestProduction;
	}

	var build = null;
	if (isExists(canQueueInfo, marginalEnergy.cheapest)) {
		build = marginalEnergy.cheapest;
	} else if (isExists(canQueueInfo, "Urban Structures") && isExists(canQueueInfo, marginalPopulation.cheapest)) {
		build = marginalPopulation.cheapest;
	} else {
		build = marginalArea.cheapest;
	}

	return build;
}

function getCost(structuresObj, conStruct, type, MarginalPopulation, MarginalArea, MarginalEnergy) {
	var cheapest;
	var cheapestPrice = -1;
	var needEnergy = 0;
	for (var aa in conStruct) {
		for (var bb in structuresObj) {
			if (structuresObj[bb].structureName == conStruct[aa]) {
				var value;
				switch (type) {
					case 1:
						value = structuresObj[bb].production;
						break;
					case 2:
						value = structuresObj[bb].structureEnergy;
						break;
						//case 3: break;
					case 4:
						value = structuresObj[bb].structurePopulation;
						break;
					case 5:
						value = structuresObj[bb].structureArea;
						break;
				}
				var temp = parseInt(structuresObj[bb].cost) / value;
				var costInqueue = temp;
				var populationPlus = 0;
				var areaPlus = 0;
				var energyPlus = 0;
				if (queueInfo[conStruct[aa]] > 0) {
					for (var i = 0; i < queueInfo[conStruct[aa]]; i++) {
						costInqueue = Math.ceil(costInqueue * 1.5);
					}
				} else {
					costInqueue = 0;
				}
				if (structuresObj[bb].structurePopulation == -1) {
					populationPlus = MarginalPopulation / value;
				}
				if (structuresObj[bb].structureArea == -1) {
					areaPlus = MarginalArea / value;
				}
				if (MarginalEnergy != null && structuresObj[bb].structureEnergy < 0) {
					energyPlus = -MarginalEnergy * structuresObj[bb].structureEnergy / value;
				}
				temp = temp + costInqueue + populationPlus + areaPlus;
				if (cheapestPrice == -1) {
					cheapestPrice = temp;
					cheapest = conStruct[aa];
				} else {
					if (temp < cheapestPrice) {
						cheapestPrice = temp;
						cheapest = conStruct[aa];
					}
				}
				break;
			}
		}
	}
	Log("type:" + type + " cheapest:" + cheapest + " cheapestPrice:" + cheapestPrice);
	return {
		cheapest: cheapest,
		cheapestPrice: cheapestPrice
	};
}

function isExists(canQueueInfo, structureName) {
	var ret = false;
	for (var aa in canQueueInfo) {
		if (canQueueInfo[aa] == structureName) return true;
	}
	return ret;
}

function getLevelFromStructuresObj(structuresObj, structureName, queueInfo) {
	var v1 = 0;
	var v2 = 0;
	for (var aa in structuresObj) {
		if (structuresObj[aa].structureName == structureName) {
			v1 = structuresObj[aa].structureLevel;
			break;
			//return structuresObj[aa].structureLevel;
		}
	}
	for (var bb in queueInfo) {
		//console.log(bb + "  " +queueInfo[bb]);
		if (bb == structureName) {
			v2 = queueInfo[bb];
		}
	}
	var ret = parseInt(v1) + parseInt(v2);
	//console.log(structureName + "  " + v1+ "  " +v2 + "  " + ret);
	return ret;
}

function getFormFromStructuresObj(structuresObj, structureName) {
	//console.log(structureName);
	var ret = 0;
	for (var aa in structuresObj) {
		if (structuresObj[aa].structureName == structureName) {
			//console.log(structuresObj[aa].structureName);
			//console.log(structuresObj[aa].structureForm.innerHTML);
			return structuresObj[aa].structureForm;
		}
	}
	return ret;
}

prepareBaseLocs();
//getBaseLocs(prepAutoBuild);
