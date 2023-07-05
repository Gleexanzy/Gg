var config = {
};

log('simulation begins here')
var config = {
    ScriptTitle: {
        type: 'noop',
        label: ' bustabit passive '
    },
    baseBet: {
        value: 2000,
        type: 'balance',
        label: 'beginning bet'
    },
 
    stop: {
        value: 300000,
        type: 'balance',
        label: 'restart betting if bet over'
		
    },
    poSettings: {
        type: 'noop',
        label: 'payout config'
    },
    basePayout: {
        value: 1.26,
        type: 'multiplier',
        label: 'begin payout'
    },
    lossPoSettings: {
        value: 'fixPo',
        type: 'radio',
        label: 'LOSE payout config',
        options: {
            fixPo: {
                value: 1.69,
                type: 'multiplier',
                label: 'loss payout:'
            },
            increasePo: {
                value: 1,
                type: 'multiplier',
                label: 'increase payout by +'
            },
            decreasePo: {
                value: 1,
                type: 'multiplier',
                label: 'lower payout by -'
            }
        }
    },
    lossPoSettinginfo: {
        type: 'noop',
        label: 'increase and lower 1 = 0.01'
    },
    lossBetSettings: {
        value: 'increaseBet',
        type: 'radio',
        label: 'LOSE bet config',
        options: {
            fixBet: {
                value: 2,
                type: 'multiplier',
                label: 'fix fold bet'
            },
            increaseBet: {
                value: 0,
                type: 'multiplier',
                label: 'increase fold by +'
            },
            decreaseBet: {
                value: 1,
                type: 'multiplier',
                label: 'decrease fold by -'
            }
        }
    },
    resetXLossesWaitForGreen: {
        value: 2,
        type: 'number',
        label: 'Reset/Wait for Greens After X Losses'
    },
    greensToWait: {
        value: 1,
        type: 'number',
        label: 'number of greens to wait for'
    }
 
};


var lossCounter = 0;
var activateGames=0;
var bettingActive = true;

log('bustanut has started');

var currentBet = config.baseBet.value;
var currentPayout = config.basePayout.value
var baseFoldPo = config.lossBetSettings.options.fixBet.value;

engine.bet(roundBit(currentBet), currentPayout);
log('Begining game Bet:', currentBet / 100, 'bits, Payout', currentPayout);

engine.on('GAME_STARTING', onGameStarted);
engine.on('GAME_ENDED', onGameEnded);

function onGameStarted() {
    if(bettingActive)
    {
        engine.bet(roundBit(currentBet), currentPayout);
        log('next bet:', currentBet / 100, 'bits, Payout:', currentPayout)
    }
}

function onGameEnded() {
    var lastGame = engine.history.first()

    if (!lastGame.wager) {
    
       if(lastGame.bust >= 2)
       {
            activateGames++;

            if(activateGames == config.greensToWait.value)
            {
              bettingActive = true;
              log('Enabling Betting. Activation hit.');
            }
          
            return;
       }
       else
       {
            log('RESET WAIT.');
            activateGames = 0;
            return;
       }

    }

    

    // Win
    if (lastGame.cashedAt) 
    {
        currentBet = config.baseBet.value;
        currentPayout = config.basePayout.value;
        baseFoldPo = config.lossBetSettings.options.fixBet.value;
        log('WIN, next bet:', currentBet / 100, 'bits, Payout', currentPayout)
        lossCounter=0;
    } 
    else 
    {
        lossCounter++;
        // Loss
        if (config.lossPoSettings.value === 'fixPo') {
            currentPayout = config.lossPoSettings.options.fixPo.value;
            log('LOSE, next payout changed to', currentPayout)
            if (config.lossBetSettings.value === 'fixPo') {
                currentBet = Math.ceil(currentBet * baseFoldPo / 100) * 100;
            } else {
                if (config.lossBetSettings.value === 'increaseBet') {
                    baseFoldPo = baseFoldPo + (config.lossBetSettings.options.increaseBet.value / 100);
                    currentBet = Math.ceil((currentBet * baseFoldPo)/100)*100;
                } else {
                    console.assert(config.lossBetSettings.value === 'decreaseBet');
                    baseFoldPo = baseFoldPo + (config.lossBetSettings.options.decreaseBet.value / 100);
                    currentBet = Math.ceil((currentBet * baseFoldPo)/100)*100;
                }
            }
        } else {
            if (config.lossPoSettings.value === 'increasePo') {
                currentPayout += config.lossPoSettings.options.increasePo.value / 100;
                log('LOSE, Next payout changed to', currentPayout)
                if (config.lossBetSettings.value === 'fixPo') {
                    currentBet = Math.ceil(currentBet * baseFoldPo / 100) * 100;
                } else {
                    if (config.lossBetSettings.value === 'increaseBet') {
                        baseFoldPo = baseFoldPo + (config.lossBetSettings.options.increaseBet.value / 100);
                        currentBet = Math.ceil((currentBet * baseFoldPo)/100)*100;
                    } else {
                        console.assert(config.lossBetSettings.value === 'decreaseBet');
                        baseFoldPo = baseFoldPo + (config.lossBetSettings.options.decreaseBet.value / 100);
                        currentBet = Math.ceil((currentBet * baseFoldPo)/100)*100;
                    }
                }
            } else {
                console.assert(config.lossPoSettings.value === 'decreasePo');
                currentPayout -= config.lossPoSettings.options.decreasePo.value / 100;
                log('LOSE, Next payout changed to', currentPayout)
                if (config.lossBetSettings.value === 'fixPo') {
                    currentBet = Math.ceil(currentBet * baseFoldPo / 100) * 100;
                } else {
                    if (config.lossBetSettings.value === 'increaseBet') {
                        baseFoldPo = baseFoldPo + (config.lossBetSettings.options.increaseBet.value / 100);
                        currentBet = Math.ceil((currentBet * baseFoldPo)/100)*100;
                    } else {
                        console.assert(config.lossBetSettings.value === 'decreaseBet');
                        baseFoldPo = baseFoldPo + (config.lossBetSettings.options.decreaseBet.value / 100);
                        currentBet = Math.ceil((currentBet * baseFoldPo)/100)*100;
                    }
                }
            }
        }

        //loss section still implement wait tech.

        if(lossCounter % config.resetXLossesWaitForGreen.value == 0)
        {
           bettingActive = false;
        }
    }


    if (currentBet > config.stop.value) {
        log(currentBet, 'Maximum amount reached, Script restarting.');
		currentBet = config.baseBet.value;
		currentPayout = config.basePayout.value;
		baseFoldPo = config.lossBetSettings.options.fixBet.value;
    }
}

function roundBit(bet) {
    return Math.round(bet / 100) * 100;
}

