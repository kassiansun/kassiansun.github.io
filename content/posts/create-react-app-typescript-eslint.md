---
title: 'Quick Start Guide with create-react-app (TypeScript + Eslint + Prettier)'
date: 2021-12-02T12:13:43+08:00
---

## Why use `create-react-app`

Originally I was using a manually written webpack script to build my react project. The reason
was to reduce the length of `package-lock.json`. But as I'm diving deeper into the frontend ecosystem,
it doesn't look like a good choice to wait for frontend guys fixing the dependency hell. So instead
of waiting for a neat solution, I need to find a quick solution.

As part of the lessons I learned from handwriting webpack scripts, it's hard to build a complete
and reusable scaffold to build a React.js + TypeScript + ESLint project. `create-react-app` is good
enough with only minimal extra configurations. This blog is mainly a note on how to quickly start
another project later.

## Create the project

1. Create an empty project with TypeScript: `npx create-react-app mproj --template typescript`
2. Add essential dependencies for my favorite code style tools:
`npm install --save-dev @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-config-airbnb eslint-config-airbnb-typescript eslint-config-prettier eslint-plugin-jest prettier`

## add eslintConfig in package.json

Basically, it's airbnb style plus prettier style, so I can use `prettier` to quickly format the code.
The extra rules are to resolve the style warnings I've encountered during development.

```
  "eslintConfig": {
    "extends": [
      "airbnb",
      "airbnb-typescript",
      "plugin:jest/recommended",
      "prettier"
    ],
    "plugins": [
      "react",
      "@typescript-eslint",
      "jest"
    ],
    "env": {
      "browser": true,
      "es6": true,
      "jest": true
    },
    "parser": "@typescript-eslint/parser",
    "parserOptions": {
      "project": "./tsconfig.json"
    },
    "rules": {
      "react/function-component-definition": [
        2,
        {
          "namedComponents": "arrow-function",
          "unnamedComponents": "arrow-function"
        }
      ]
    }
  },
```

## Add more constraints to tsconfig.json

I don't like the loose rules TypeScript shipped by default, since we chose TypeScript, do it
more aggressively.

```
    "noImplicitAny": true,
    "noImplicitThis": true,
    "noImplicitReturns": true
```

## The old hand-written webpack.js

It's hard to say the old script was neat and easy to maintain, but it's a good reminder
on how webpack works beneath the `react-scripts`.

```
"start": "webpack --watch --mode=development --config scripts/webpack.js",
"build": "webpack --mode=production --config scripts/webpack.js"
```

```
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ESLintPlugin = require('eslint-webpack-plugin');

module.exports = {
  entry: {
    main: './src/index.tsx',
  },
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, '../public'),
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.json'],
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'public/index.tmpl.html',
    }),
    new ESLintPlugin(),
  ],
  module: {
    rules: [
      {
        test: /\.svg$/,
        loader: 'svg-inline-loader',
      },
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
      },
      {
        test: /\.(ts|tsx)$/,
        exclude: /node_modules/,
        loader: 'ts-loader',
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
};
```
