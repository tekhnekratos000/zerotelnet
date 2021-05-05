// Wikipedia telnet server
//
// To install depenedncies:
// npm install request
// npm install cheerio
//

var net = require( 'net' );
var cheerio = require('cheerio');
var request = require('request');
var fs = require('fs');

var port = 1081;
var logo = fs.readFileSync( 'wiki-logo.txt' );
var baseUrl = 'http://en.wikipedia.org/wiki/';
var ps1 = '\n\n>>> ';

function recv( socket, data ) {

    data = data.toString().replace( /(\r\n|\n|\r)/gm, '' );
    data = data.replace( /\s/g, '_' );

    if ( data === 'quit' ) {
        socket.end( 'Bye!\n' );
        return;
    }

    request( { uri: baseUrl + data }, function ( error, response, body ) {
        if ( body && body.length ) {
            $ = cheerio.load( body );
            socket.write( $( '#mw-content-text p' ).first().text() + '\n' );
        } else {
            socket.write( 'Error: ' + response.statusCode );
        }
        socket.write( ps1 );
    } );
}
 
net.createServer( function ( socket ) {
    socket.write( logo );
    socket.write( ps1 );
    socket.on( 'data', recv.bind( null, socket ) );
} ).listen( port );
