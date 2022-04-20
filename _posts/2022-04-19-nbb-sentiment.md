---
layout: default
title: "nbb sentiment"
date: 2022-04-19
categories: code
tags: nbbjs gcp
---

[nbb](https://github.com/babashka/nbb) brings clojure scripting to nodejs environments.  Although we've
been building clojurescript programs for nodejs for a while now, `nbb` is different.

* cljs files are never compiled to js (nbb is an interpreter).  If you're making a serverless function, or a script, the cljs files are packaged.
* macros are not evaluated at compile time, as in clojurescript. This feels more natural if you're accustomed to clojure.
* easy to `require` and use npm packages
* simple and intuituive repl experience

I decided to try this out with a `@google-cloud` module.  Create an npm project with the following.

```bash
npm install -g nbb
mkdir nbb-test
cd nbb-test
npm init -y
npm install @google-cloud/language
npx nbb nrepl-server
```

Start an editor and connect your repl.  Create an `app.cljs` file with the contents below and evaluate the namespace.

```clj
(ns app
  (:require [clojure.pprint :refer [pprint]]
            [promesa.core :as p]
            ["@google-cloud/language" :as language]))

(def language-client (new (.-LanguageServiceClient language)))

(p/catch
  (p/then 
    (.analyzeSentiment 
      language-client 
      #js {:document #js {:content "I am not angry" :type "PLAIN_TEXT"}}) 
    #(pprint (js->clj %)))
  #(println %))
```

This will fail if your nodejs session can not authenticate.  The `@google-cloud/language` module
will search your local environment for `application-default` credentials, so it's straight forward to
login using gcloud. Once you've logged in, you can evaluate your namespace above again, and you should
see a sentiment analysis result from GCP.

```bash
gcloud auth application-default login
npx nbb app.cljs
```

This assumes that the Natural Language apis are enabled in your current gcloud google project.

Incidentally, this is the same as running the following gcloud command.

```bash
gcloud ml language analyze-sentiment --content="I am not angry" 
```

