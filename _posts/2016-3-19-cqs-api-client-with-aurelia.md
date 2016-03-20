---
layout: post
title: A CQS Api Client With Aurelia and TypeScript
category: Aurelia
---

Every SPA needs a way to communicate with the server and Aurelia features a `HttpClient` that implements the [Fetch API](https://fetch.spec.whatwg.org/). But this client is a bit to low level for my taste, I want my app to not be concerned with _how_ a client communicates with the server. I want my app to take a Command/Query approach i.e when something needs to change I'll send a command request and when I just want some data but I don't want to change anything then I'll send a query request.

On my server, I'm using the following conventions

* For commands: `[POST] baseUrl/{command}` . For every request I'm returning a `CommandResult` which looks like this
```javscript
export interface ICommandResult {
    errors: any;
    hasErrors: boolean;
    data: any;
}
``` 
* For queries: `[GET] baseUrl/{query}`. It returns the query result, usually an object or a list of objects. 

So, my API client interface looks like this 

```javascript
interface IApiClient{
    
     execute(cmd: string, data: any, func: (res: ICommandResult) => void);
     query<T>(query: string, func: (result: T) => void, params?: any);
}

```

Considering the http client returns a promise and I want to handle request errors in one place, I've decided to process any result inside a function that will be automatically invoked by the API client, that's why I'm always passing a function when invoking `execute` or `query`.

Let's see some implementation details

```javascript
  execute(cmd: string, data: any, func: (res: ICommandResult) => void) {

        return this.http.fetch(cmd, {
            method: "POST",
            body: json(data)
        })
            .then(resp => resp.json().then(d=> func(d)))
            .catch(er=> this.handleError(er));

    }
``` 
Pretty straightforward, I'd say.  We send the command as a POST in JSON format, receive a response which needs to be processed as a json, then we invoke the result processor, while any errors will be handled inside `handleError` (will see its implementation a bit later). Any command result processor will check for validation messages(the `hasError` field) and has the error messages available in the `errors` field, and it can get things like an id or something generated on the server side from that `data` field.

If you know the original CQS (which applies to objects), returning a command result with some data looks like a violation of the principle. However, this is not _that_ CQS and we'd only complicate things if we'd have to query specifically for that data later. 

Here's an example of how to use it

```javascript
 this.api.execute(/*command name */"createAsset"
                 ,/* command params*/ { name:"test" }
                 ,/* result processor */ r => {
                    if (r.hasErrors) {
                        //this will be used by an <errors> element
                        this.error = r.errors.name;
                        return;
                    }
                    //do something with the new id
                    console.log("new id:"+r.data.id); 
                });

```   

The `query` is similar, only that we return a typed result

```javascript
 query<T>(query: string, func: (result: T) => void, params?: any) {
    
        if (params) {
            var args = $.param(params);
            query = query + "?" + args;
        }

        return this.http.fetch(query)
            .then(r => r.json().then(b => func(b)))
            .catch(err => this.handleError(err));

    }

```
I'm using jquery to create the query string. The query result is considered to be of the specified `<T>` type.

```javascript
this.api.query<MyResult>("getapicommanddata", result => {
           // do something with the result  
         this.data=result;
       });

```
Plain and simple.

Here's the full ApiClient class in all its glory
```javascript

import {HttpClient, HttpClientConfiguration,json} from "aurelia-fetch-client";
import 'fetch'; //polyfill
import * as $ from 'jquery';

export interface ICommandResult {
    errors: any;
    hasErrors: boolean;
    data: any;
}


export abstract class ApiClient {
    constructor(private http: HttpClient) {
        this.http.configure((cf: HttpClientConfiguration) => {
            cf.useStandardConfiguration()
                .defaults.headers = {
                    'Accept': 'application/json',
                    'X-Requested-With': 'Fetch'
                };


        });

    }

    configure(baseUrl: string) {
        this.http.baseUrl = baseUrl + (baseUrl.endsWith("/") ? "" : "/");
    }




    query<T>(query: string, func: (result: T) => void, params?: any) {
    
        if (params) {
            var args = $.param(params);
            query = query + "?" + args;
        }

        return this.http.fetch(query)
            .then(r => r.json().then(b => func(b)))
            .catch(err => this.handleError(err));

    }

    protected handleError(err) {
        
        //todo send to raygun
        console.debug("Server err. ");
        console.debug(err);
    }

    execute(cmd: string, data: any, func: (res: ICommandResult) => void) {

        return this.http.fetch(cmd, {
            method: "POST",
            body: json(data)
        })
            .then(resp => resp.json().then(d=> func(d)))
            .catch(er=> this.handleError(er));

    }


}

```

The code is more maintainable and allows us to change how we communicate with the server in the future. The class is abstract because I want to be able to talk to multiple endpoints (on the same server) so I'd create a class for each endpoint and optionally, semantic methods that makes the client look like a normal app service or even a repository.

```javascript

@autoinject
export class FooApi extends ApiClient {
    constructor(http: HttpClient) {
        super(http);
        
        //setup base api url
        this.configure("api/foo-endpoint");
    }

    //override error handling
    protected handleError(err) {
      
        Notifier.instance.error();
    }

    //a nice facade for the command
    delete(item: IdName, success: Function) {
        this.execute("deleteFoo", {assetId: item.id }, r=> success(r));
    }
}

```
 
As a trivia fact, when I've started building my app, Aurelia's HttpClient was a wrapper over XMLHttpRequest, but now it implements the FetchApi. Since I was using an abstraction like the above, I just needed to change the `ApiClient` implementation and nothing else. But I recommend using this kind of abstraction primarily for developer friendliness as we don't need to change how we communicate with the server very often. But it's a nice side effect.

While I'm still a novice when it comes to developing single page apps, I can't help but notice how often I can use the same approaches I'm using on the server side. In the end,we're still building an app so all the design patterns and SOLID stuff applies, but we're using Aurelia/TypeScript instead of WPF/C#.  
 

   