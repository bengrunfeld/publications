# Creating an React, Express, & Webpack Application

This app boilerplate is built on top of [Expack](https://github.com/bengrunfeld/expack). The instructions of how to get to the starting point needed for this article can be found [here](https://medium.com/@bengrunfeld/...)

## What we want to build

An Express and Webpack configuration that supports a React application and transpiles JSX, as well as providing React development tools like React-specific Linting and Unit Testing for React components. 

## Tech Stack (in addition to Expack's stack)

* enzyme (used with Jest)
* enzyme-adapter-react-16
* eslint-plugin-react

# Step 1: Install React

Either you can clone Expack or follow the steps in the [previous article]() to get to the starting position. 

    npm install --save react react-dom
    npm install --save-dev babel-preset-react

Edit `.babelrc` and make it look like this:

    {
      'presets': ['env', 'react']
    }

Change `./src/index.js` to look like this:

    import React from 'react'
    import ReactDOM from 'react-dom'
    import Message from './js/Message'
    
    import './css/style.css'
    
    ReactDOM.render(
      <Message />,
      document.getElementById('react-container') // eslint-disable-line no-undef
    )
    
    // Needed for Hot Module Replacement
    if(typeof(module.hot) !== 'undefined') { // eslint-disable-line no-undef  
      module.hot.accept()                    // eslint-disable-line no-undef  
    }

Don't put any other React components in this file, because Express-Webpack HMR will throw an error if you do. Put them all in other files and import them.

You can delete `adder.js` and `adder.test.js` from the previous tutorial.

Create the following file `./src/js/Message.js` with the following code:

    import React from 'react'
    
    const Message = () => {
      return (
        <div className="content">
          <h1>Rexpack</h1>
          <p className="description">React, Express, and Webpack Boilerplate Application</p>
          <div className="awful-selfie"></div>
        </div>
      )
    }
    
    export default Message

Now you use `npm run buildDev` and `npm start`, and then navigate to `localhost:8080` in your browser to test.

Notice that you receive some linting errors about `React` not being defined, or about being defined but not used. We'll fix those in a sec.

# Step 2: Update ESLint to Lint React and JSX

First install the React plugin for ESLint:

    npm install --save-dev eslint-plugin-react

Now update `.eslintrc.js` to the following:

    module.exports = {
      "plugins": [ "react" ],
      "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
      ],
      "parser": "babel-eslint"
    };

If you run `npm run buildDev` now, you'll see that all the React linting errors have gone. 

To REALLY test that it works, let's go and break something! In `Message.js`, change one of the `className`s to just `class`. Save it and you should see a linting error in the terminal window if you have HMR running. If you don't, just run `npm run buildDev` and it will pop right up. Change it back to `className` and the linting error will be resolved.

# Step 3: Update Jest to use Enzyme to Unit Test React Components

First let's install Enzyme and its adapter for React 16:

    npm install --save-dev enzyme enzyme-adapter-react-16

Now add a file to the root called `.jest.config.js` with the following code:

    module.exports = {
      setupFiles: ['<rootDir>/src/test/setup.js']
    }

Create the file `./src/test/setup.js` with the following code:

    import Enzyme from 'enzyme'
    import Adapter from 'enzyme-adapter-react-16'
    
    Enzyme.configure({ adapter: new Adapter() })

Create the file `./src/js/test/Message.test.js` with the following code:

    import React from 'react'
    import Enzyme, { shallow, mount, render } from 'enzyme'
    import Message from '../Message'
    
    describe('<Message />', () => {
      test('renders a single <p> tag', () => {
        const wrapper = shallow(<Message />);
        expect(wrapper.find('p')).toHaveLength(1);
      });
    })

Now if you run `npm test` (script was set up in the previous tutorial), the tests should pass.

And there you have it folks! React has been set up with Linting and Unit Testing inside of an Express-Webpack application. 


