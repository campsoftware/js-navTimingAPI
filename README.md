# js-navTimeingAPI

### Summary

Logs to console.log detailed timing metrics related to the navigation and load of a web page.

### console.log

```javascript
[Log] 1.8330s Total Time (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4003)
[Log]     0.1270s DNS Lookup (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4004)
[Log]     0.1020s TCP Connection (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4005)
[Log]     0.0550s TLS Handshake (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4006)
[Log]     0.6040s Server Total (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4007)
[Log]         0.3066s Nginx, etc: (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4008)
[Log]         0.2974s Xanadu (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4009)
[Log]     0.0480s Content Download (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4010)
[Log]     0.9510s Client Render (F30BB9DD-DE9A-4F13-91B1-79BA4A776585, line 4011)
```

### PHP Init Begin Time

```php
$xanTimeBegin = \microtime( true );

function microsecsDiff( $pMicrosecs ): string {
	return \round( ( \xan\microsecsNow() - $pMicrosecs ), 4 ) . "s";
}

// Do Stuff
```

### Javascript with PHP End Time

```javascript
window.phpExecutionTime = '<?= \xan\microsecsDiff( $xanTimeBegin ); ?>';
window.addEventListener('load', () => {
    // Get Navigation Timing
    const [navigation] = performance.getEntriesByType('navigation');

    if (navigation) {
        const dnsLookupTime = ((navigation.domainLookupEnd - navigation.domainLookupStart) / 1000).toFixed(4) + 's';
        const tcpConnectTime = ((navigation.connectEnd - navigation.connectStart) / 1000).toFixed(4) + 's';
        const tlsHandshakeTime = navigation.secureConnectionStart > 0
            ? ((navigation.connectEnd - navigation.secureConnectionStart) / 1000).toFixed(4) + 's'
            : '0.0000s'; // Only for secure connections
        const serverSideExecutionTimeSecs = (navigation.responseStart - navigation.requestStart) / 1000;
        const serverSideExecutionTime = serverSideExecutionTimeSecs.toFixed(4) + 's';
        const contentDownloadTime = ((navigation.responseEnd - navigation.responseStart) / 1000).toFixed(4) + 's';

        // Include client-side processing in Total Time
        const totalNavigationTime = ((navigation.domComplete - navigation.startTime) / 1000).toFixed(4) + 's';
        const clientSideProcessingTime = ((navigation.domComplete - navigation.responseEnd) / 1000).toFixed(4) + 's';

        // Drop the `s` from `window.phpExecutionTime` and convert to milliseconds for calculations
        const phpExecutionTimeMs = parseFloat(window.phpExecutionTime.replace('s', '')) * 1000;
        const serverOverheadTimeSecs = (serverSideExecutionTimeSecs - (phpExecutionTimeMs / 1000)).toFixed(4) + 's';

        // Log the timings breakdown with client-side processing included in total time
        console.log(`${totalNavigationTime} Total Time`);
        console.log(`    ${dnsLookupTime} DNS Lookup`);
        console.log(`    ${tcpConnectTime} TCP Connection`);
        console.log(`    ${tlsHandshakeTime} TLS Handshake`);
        console.log(`    ${serverSideExecutionTime} Server Total`);
        console.log(`        ${serverOverheadTimeSecs} Nginx, etc:`);
        console.log(`        ` + window.phpExecutionTime + ` Xanadu`);
        console.log(`    ${contentDownloadTime} Content Download`);
        console.log(`    ${clientSideProcessingTime} Client Render`);

        // Message Display
        xanMessageDisplay( '<?= FI_PSOS_STOPWATCH ?>&nbsp;' + `Loaded Page ${totalNavigationTime} [${serverSideExecutionTime}]`, false );
    } else {
        console.warn('Time Log is not available via Navigation Timing API.');
    }
});
```
