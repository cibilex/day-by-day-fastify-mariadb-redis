# Fastify
- Everything is a pluin with fastify.
  
```js
import Fastify from "fastify"; 

const app = Fastify({ logger: true });


app.get('/', (req, res) => res.send('hi world'));
const start = async () => {
    try {
        const address = await app.listen({ port: 8080 }) // alternate of app.listen({logger:true},(err,address)=>)
        console.log('app listening on' + address);
    } catch (error) {
        app.log.error('something went wrong', error);
        process.exit(1);
    }
}
start()
```

- this is FastifyInstance in routers .using an arrow function will break the binding of this to the FastifyInstance.

- **listen(options)**: 
      1. ***host***:   default 127.0.0.1
 
            |   value   | IPV4  | IPV6  |
            | :-------: | :---: | :---: |
            | localhost |  yes  |  yes  |
            | 127.0.0.1 |  yes  |  no   |
            |    ::     |  yes  |  yes  |
            |    ::1    |  no   |  yes  |


- **Plugins** : `fastify.register(require('my-plugin'),[options])`
    - options: 
        1. `prefix` : prefix for all routes in current plugin
        2. `logLevel`('warn','debug')
    - **[fastify-plugin]** : used to access the variable we created via plugins .Any usage of the instance will behave the same as it would if called within the plugins function i.e. if decorate is called, the decorated variables will be available within the plugins function unless it was wrapped with fastify-plugin.
        - ```js
            app.register(fp(function (app, opts, done) {
            app.decorate('hi', 'hi world from plugins')
            done()
            }, { fastify: '4.x' // min fastify version to register this plugin
            , name: 'my-first-plugin',
             dependencies: ['plugin1', 'plugin-2'] // which plugins requested to run this plugin
              }))

            app.get('/', async (req, res) => {
            return app.hi
            })

          ```
    - We can not access a decorator before registration.Order of registration is important for dependencies .we can not access an dependencies before registration.
    - app.after: after registeration, app.ready() all registeration declared.Use them with await to listen server.
- **Decorators**  : to avoid to deoptimization of javascript engine use decorateRequest before add key to request.initialize decorator '' if  decorator object will be string,initialize null if decorator will be object or function.
    - to adding value to reply.use `decorateReply`
    - ```js
        fastify.decorateRequest('user', '')
        fastify.decorateRequest('hi', null)

        fastify.addHook('preHandler', (req, reply, done) => {
        req.user = 'Bob Dylan'
        req.hi=()=>'hi worlf from prehandler'
        done()
        })
        // And finally access it
        fastify.get('/', (req, reply) => {
        reply.send(`Hello, ${req.user}!`)
        })
      ```
    - **decorate(name,val)**: `app.decorate('myDb',()=>hi from db)` - `app.decorate('myVal','hi from decorate')`.Thus we can access variables via fastify. app.myDb() ,app.myVal 
    - `hasDecorator` - `hasRequestDecorator` - `hasReplyDecorator`

- **ROUTES**: routes are endpoints of application.We can declare an route with 2 ways.     
       - *options*: 
           - `method`: GET POST PUT DELETE PATCH SEARCH  
           -  `schema`:  ajv validation for payload. `body` `response` `query or querystring` `params`     
           -  `bodyLimit`: max body limit in bytes.default 1Mib.    
           -  `logLevel`: set log level. 'warn' 'debug'  
           -  `prefixTrailingSlash`: How to handle path. with '/' or withou.**just** for prefix!. `both`(default) `slash` `no-slash`
           -  `url`: path    
        -  `short declaration`: app[method]('url',[options],handler) : app.get('/',(req,res)=>void 0).   
            - short declaration is supported to all method.   
       - *url building* : an url can be static,parametric ,with wilcard or with regexp. `/user/:name` `/user/*` `/user/:name-:username`   
       -  If we use async handler ,we don't need to use reply.send just return data.`handler: async (req, reply) => 'hi world',`  
        - we can not return undefined!
       -  Also we can wait for reply.  
            - ```js
             
                    handler: async (req, reply) => {
                    setTimeout(() => {
                    reply.send('thans for waiting')
                    }, 2500)

                    await reply
                    }
                  ```      


- fastify decorateRequest kullanın diyo ama req.user kullanılmamış 
- authentication işlemleri prehandlerda yapılmış ama fastify preValidationda yapın diyor.





























[fastify-plugin]: https://www.npmjs.com/package/fastify-plugin
