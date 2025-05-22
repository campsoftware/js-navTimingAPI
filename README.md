# js-navTimingAPI

### Summary

Logs to console.log detailed timing metrics related to the navigation and load of a web page.

### console.log

```text
[Log] 1.8330s Total Time
[Log]     0.1270s DNS Lookup
[Log]     0.1020s TCP Connection
[Log]     0.0550s TLS Handshake
[Log]     0.6040s Server Total
[Log]         0.3066s Nginx, DNS, etc
[Log]         0.2974s PHP
[Log]     0.0480s Content Download
[Log]     0.9510s Client Render
```

### PHP Init Begin Time

```php
$xanTimeBegin = \microtime( true );

function microsecsDiff( $pMicrosecs ): string {
	return \round( ( \xan\microsecsNow() - $pMicrosecs ), 4 ) . "s";
}

// Do Stuff
```

### Javascript with PHP Total Time

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
        console.log(`        ${serverOverheadTimeSecs} Nginx, DNS, etc:`);
        console.log(`        ` + window.phpExecutionTime + ` PHP`);
        console.log(`    ${contentDownloadTime} Content Download`);
        console.log(`    ${clientSideProcessingTime} Client Render`);

        // Message Display
        xanMessageDisplay( '<?= FI_PSOS_STOPWATCH ?>&nbsp;' + `Loaded Page ${totalNavigationTime} [${serverSideExecutionTime}]`, false );
    } else {
        console.warn('Time Log is not available via Navigation Timing API.');
    }
});
```
