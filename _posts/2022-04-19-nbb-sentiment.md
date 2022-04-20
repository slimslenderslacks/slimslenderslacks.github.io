---
layout: default
title: "nbb sentiment"
date: 2022-04-19
categories: code
tags: nbbjs gcp
---

[nbb](https://github.com/babashka/nbb) brings clojure scripting to nodejs environments.  We've been building clojurescript programs for nodejs for sometime, but `nbb` is interesting for a few reasons.

* cljs files do not get compiled to js (nbb is an interpreter).  If you're making a serverless function, or a script, the cljs files are distributed directly.
* macros are not evaluated at compile time, as in clojurescript, so macros feel more natural
* really nice support for working with the npm ecosystem
* a simple and intuituive repl experience

I decided to see what this would look like with a `@google-cloud` module.  Start an npm project with the following.

```bash
npm install -g nbb
mkdir nbb-test
cd nbb-test
npm init -y
npm install @google-cloud/language
npx nbb nrepl-server
```

Start an editor and connect your repl.  Evaluate the following namespace.  Create an `app.cljs` file with the following contents.

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
      #js {:document #js {:content "I am angry" :type "PLAIN_TEXT"}}) 
    #(pprint (js->clj %)))
  #(println %))
```

This will fail if your nodejs session can not authenticate.  However, the `@google-cloud/language` module will search your local environment for `application-default` credentials.  You can create an `application-default` login using gcloud.

```bash
gcloud auth application-default login
npx nbb app.cljs
```

You should see a sentiment analysis performed on your document.  This is not at different from just running the following.

```bash
gcloud ml language analyze-sentiment --content="I am angry" 
```

