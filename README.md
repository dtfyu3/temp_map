### Interactive countries annual temperature map ###

_.mbtiles file was generated from large .geojson file with [this tool](https://github.com/mapbox/tippecanoe)_

You need running tileserver-gl server with your .mbtiles file.
To run tileserver-gl you need to install it via npm and create config.json file like this:
```
{
    "options": {
        "headers": {
            "Access-Control-Allow-Origin": "*"
        }
    },
    "data": {
        "localized_with_temps": {
            "mbtiles": "localized_with_temps.mbtiles"
        }
    }
}
```
Then you run this server from directory where you've created config.json using:
```
tileserver-gl --config config.json
```
Note that may throw CORB warning when you try to access it from your code directly. To avoid this you should create proxy server which will transfer given requests to tileserver-gl server itself. 
For example like this:
```
app.use('/tiles', (req, res, next) => {
    const cacheKey = req.originalUrl;
    const cachedResponse = cache.get(cacheKey);

    if (cachedResponse) {
        res.set(cachedResponse.headers);
        res.send(cachedResponse.body);
    } else {
        createProxyMiddleware({
            target: 'http://localhost:8080',
            changeOrigin: true,
            pathRewrite: { '^/tiles': '' },
            onProxyRes: (proxyRes, req, res) => {
                const body = [];
                proxyRes.on('data', (chunk) => body.push(chunk));
                proxyRes.on('end', () => {
                    const response = {
                        headers: proxyRes.headers,
                        body: Buffer.concat(body)
                    };
                    cache.set(cacheKey, response);
                });
            }
        })(req, res, next);
    }
});
app.listen(3000, () => {
    console.log('Прокси-сервер запущен на http://localhost:3000');
});
```
And you simply replace link here from ```localhost:8080``` to ```localhost:port_that_your_proxy_is_on```:
```
 const vectorTiles = L.vectorGrid.protobuf(
        'http://localhost:3000/tiles/data/YOUR_NAME/{z}/{x}/{y}.pbf', {
        rendererFactory: L.canvas.tile,
        interactive: true,
        ...
        ...
```
