function getPSR( pid, callback ) {
    var exec = require('child_process').execSync;

    var command = 'ps -A -o pid,psr -p ' + pid + ' | grep ' + pid + ' | grep -v grep |head -n 1 | awk \'{print $2}\'';

    var result = exec( command );
    
    return result.toString("utf-8").trim();
}

function getTIDs( pid ) {
    var exec = require('child_process').execSync;

    var command = 'ps -mo tid,psr -p ' + pid + ' | grep -v grep | awk \'/[0-9]/ {print("{\\042id\\042 : ", $1, ",\\042psr\\042:", $2, " }," )}\'';
    var tids = '[ ' + exec(command) + '{"id": false} ]';

    return JSON.parse(tids);
}

function setPSR( pid, psr ) {
    var exec = require('child_process').execSync;
    var command = 'taskset -pc ' + psr + ' ' + pid  + '; kill -STOP ' + pid + '; kill -CONT ' + pid;
    var result = exec(command);

    return result.toString("utf-8").trim();
}

function setTIDsPSR( pid, psr ) {
    var tids = getTIDs(pid);
    console.log(tids);
    for (var i in tids) {
	if (tids[i].id) {
	    console.log( setPSR( tids[i].id, psr ) );
	}
    }
}
