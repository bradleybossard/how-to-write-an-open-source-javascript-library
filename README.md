# Quick guide to publishing on NPM

### Links
* [Data-Driven Tests in JavaScript Using Mocha - Alex Booker](https://booker.codes/data-driven-tests-in-javascript-using-mocha/)
* [Automating Javascript Testing, Deploy with npm & Travis CI to Github (part 3) – Oren Farhi – Thoughts On Javascript and Development](http://orizens.com/wp/topics/automating-javascript-testing-deploy-with-travis-ci-part-3/)
* [inikulin/publish-please - Alternative to semantic-release ?](https://github.com/inikulin/publish-please)
* [Design and Build Your Own JavaScript Library: Tips & Tricks](https://www.sitepoint.com/design-and-build-your-own-javascript-library/)
* [ES2015 modules to UMD transform · Babel](http://babeljs.io/docs/plugins/transform-es2015-modules-umd/)
* [Use JSDoc: Index](http://usejsdoc.org/)
* [Choose an open source license - Choose a License](http://choosealicense.com/)


### Step 1 - Create a Github repo and clone it on your machine

### Step 2 - Create an npmjs.com account

### Step 3 - Configure your npm
    npm set init-author-name 'Bradley Bossard'
    npm set init-author-email 'bradleybossard@gmail.com'
    npm set init-author-url 'http://bradleybossard.com'
    npm set init-license 'MIT'
    npm set save-exact true
    cat ~/.npmrc
    npm adduser  # Follow the steps
    npm init
    
### Step 4 - Start writing the library
* Create an index.js
* Add the following

    var whatever = require('./whatever')
    
    module.exports = {
      func1: func1,
      func2: func2(param1)
    };

### Step 5 - Push to Github

    git add -A
    git commit -m "Initial commit of library"
    git push

### Step 6 - Publish module to NPM

   npm publish
   npm info <package-name>  # Test that it worked

### Step 7 - Test out that it worked
* Go to a new test directory and type the following

    npm install <package-name>
    
* Create a new file, like index.js

    var packageName = require('<package-name>');
    
    console.log(packageName.func1());
    console.log(packageName.func2('test'));
    
### Step 8 - Git tag your release

    git tag <version>   ## <version> in package.json
    git push --tags

Now, you can go to github and describe a release from that tag as well.

## Updating a package

### Step 1 - Make a change to your project

### Step 2 - Change the package.json <version> number according to SemVer

### Step 3 - Add files to git and commit

    git add -A
    git commit -m "Meaningful commit message"
    git push

### Step 4 - Tag release

    git tag <version>   # New package.json version number
    git push --tags

### Step 5 - Publish to npm

    npm publish
    npm info <package-name>  ## Verify push went smoothly

## Publish a beta version

### Step 1 - Make changes to code

### Step 2 - Update package.json version with -beta.0 suffix, like 1.0.0-beta.0

### Step 3 - Add files to git and commit

    git add -A
    git commit -m "Meaningful commit message"
    git push

### Step 4 - Tag release

    git tag <version>   # New package.json version number
    git push --tags

### Step 5 - Publish to npm

    npm publish --tag beta
    npm info <package-name>  ## Verify push went smoothly
    
Now, in the package.json, under dist-tags, there will be a latest and beta entry

### Step 6 - Install in a project

    npm install           # Installs latest
    npm install@beta      # Installs latest beta
    
## Testing with mocha and chai

### Step 1 : Install mocha and chai

    npm install mocha chai --save-dev

### Step 2 : Add a test file, for instance index.test.js

### Step 3 : Add some simple tests

    var expect = require('chai').expect;
    var library = require('./index');
      
    describe ('libraryTest', function() {
      it('should work!', function() {
        expect(true).to.be.true;
      });
    });
  
### Step 4 : Add a test commands to the list of scripts
In package.json, add (test:single will be used for travis CI later).
"scripts" : {
  "test": "mocha src/index.test.js -w"
  "test:single": "mocha src/index.test.js"
},

## Using semantic-release

### Step 1 : Install semantic-release
    npm install -g semantic-release-cli
    
### Step 2 : Setup semantic-release-cli
    semantic-release-cli setup
    
and answer the following questions.  This creates a .travis.yml file and removes
the version from the package.json.  It's recommended to replace the "version"
entry in the package.json with something like "0.0.0-semantically-released",
otherwise npm install will give an error that your package has no version.

### Step 3 : Add a test step to .travis.yml
To ensure travis runs your tests, edit the .travis.yml to include the following


    before_script:
      - npm prune
    script:
      - npm run test:single
    after_success:
      - npm run semantic-release

### Step 4: Install commitizen

    npm install --save-dev commitizen cz-conventional-changelog

### Step 5: Add a script to run git-cz
To package.json, add

    "scripts": {
      "commit": "git-cz",
      // other scripts
    }

and at the bottom, after "devDependencies:" add

    "czConfig": {
      "path": "node_modules/cz-conventional-changelog"
    }

### Step 5: Test out commitizen

    npm run commit
    
## Adding githooks to run tests before commiting

### Step 1 : Install ghooks

    npm install --save-dev ghooks
    
### Step 2 : Add ghooks config to package.json

    "config": {
      "ghooks": {
        "pre-commit": "npm run test:single"
      }
    }

## Add Instanbul for code coverage report

### Step 1 : Add instanbul to dev dependencies

    npm install istanbul --save-dev

### Step 2 : Update package.json "test" script to

    "test" : "istanbul cover -x *.test.js _mocha -- -R spec index.test.js"
    
### Step 3 : Add coverage directory to .gitignore file to avoid checking in coverage reports

## Set up Githook for checking coverage

### Step 1 : Add a script to check coverage and call it in the githooks sections

In package.json, add the following to "scripts"

    "scripts" : {
      "check-coverage" : "istanbul check-coverage --statements 100 --branches 100 --functions 100  --lines 100"


Then in the githooks object, chain it after out test:single

    "githooks":
      "pre-commit": "npm run test:single && npm run check-coverage",
      
### Step 2: Make it part of your .travis.yml such that it needs to pass in order to release

In the file travis.yml

    script:
      - npm run check-coverage
      
## Adding code coverage reporting

### Step 1 : Sign up for a code coverage account

### Step 2 : Install the codecove.io node library

    npm install --save-dev codecov.io
    
### Step 3 : Add script to run codecove.io

In the package.json

    "scripts": {
      "report-coverage" : "cat ./coverage/lcov.info | codecov",
      
### Step 4 : Add reporting step to travis

In .travis.yml

    after_success:
      - npm run report-coverage
      
After this, run `npm run commit` to run semantic release, fire off a build on
Travis to ensure it works.

## Adding badges to your build

### Step 1 : Go to shields.io to get badges for various services
      
### Step 2 : Add the badges urls to your README.md

[![travis](link-to-shields.io-image)](link-to-travis-page)

##  Add ES6 support to project

### Step 1 : Add babel as a dev dependency

    npm install --save-dev babel-cli
    
### Step 2 : Add a build script to package.json

In package.json

    "scripts" : {
      "prebuild": "rm -rf dist && mkdir dist",
      "build" : "babel src/index.js -o dist/index.js",
      
And change

    "main": "dist/index.js"
  
 `
