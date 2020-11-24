# React-ts 프로젝트 디자인 시스템 구축하기

## 디자인 시스템이란?

> 제품을 디자인하고, 실현하고, 개발할 수 있는 모든 요소들을 그룹화하는 단일한 기준을 제공하는 시스템

### 디자인 시스템 요소

1. Style Guide

   > 특정 브랜드 또는 프로덕트에서 디자인 작업 시 지켜야하는 규칙

   - 색상
   - 아이콘
   - 여백 (margin)
   - 패딩 (padding)
   - 타이포그라피 (typography)

2) Component Library

   > 재사용 가능한 컴포넌트들 모음

   - Button
   - Modal
   - ...

> 디자인 시스템에서 가장 중요한 핵심은 참여하는 누구나 접근할 수 있도록 <b>문서화</b>가 되어 있어야 한다.

## 리액트 프로젝트에서의 디자인 시스템

리액트로 UI/UX 개발 시 손쉽게 디자인 시스템 구축을 도와주는 도구로 `storybook`, `Docz`, `StyleGuidist` 가 있다.

1. Docz
2. StyleGuidist
3. Storybook

## Storybook 활용하기

## CRA 로 React typescript 프로젝트 생성하기

```
$ yarn create react-app [project name] --typescript
```

--typescript 옵션을 통해 타입스크립트가 설정된 프로젝트가 생성된다.

## Storybook 끼얹기

```
$ npx -p @storybook/cli sb init
```

`init` 명령어는 storybook 과 관련된 dependency 들을 함께 설치해주고, `stories` 폴더를 만들어 예제 코드까지 생성해준다.

compilerOptions 의 jsx 값을React-jsx 를 react로 변경

## Typescript 환경설정

Storybook 에서 Typescript 를 사용하기 위해 추가적으로 패키지들을 추가해준다.

```
yarn add --dev babel-preset-react-app react-docgen-typescript-loader
```

**babel-preset-react-app**

CRA 에서 사용하는 babel preset 과 동일

Typescript 를 적용할 때 babel-loader 를 사용하기 위해서 설치

**react-docgen-typescript-loader**

컴포넌트 props 에서 사용된 Typescript 타입들을 추출하여 문서로 만들어주는 도구

기본적으로 typescript 로 CRA 를 실행한 경우 `tsconfig.json` 이 이미 존재한다. 혹시라도 `compilerOptions` 에`jsx` 값을 `'react'`가 아닌경우 변경하면 에러표시가 사라진다.

Storybook에서 Typescript 를 쓰기 위해서 webpack 설정을 커스터 마이징 해주어야 한다.

.storybook 경로에 webpack.config.js 파일을 생성하면 Storybook 기본 webpack 설정을 커스터마이징할 수 있다.

```javascript
module.exports = ({ config, mode }) => {
  config.module.rules.push({
    test: /\.(ts|tsx)$/,
    use: [
      {
        loader: require.resolve('babel-loader'),
        options: {
          presets: [['react-app', { flow: false, typescript: true }]],
        },
      },
      require.resolve('react-docgen-typescript-loader'),
    ],
  });
  config.resolve.extensions.push('.ts', '.tsx');
  return config;
};
```

이를 통해 ts/tsx 확장자에 대해 babel-loader , react-docgen-typescript-loader 를 사용하도록 설정할 수 있다.

ts 파일에서 mdx 확장자 파일을 불러올 때 모듈이 없다는 에러를 방지하기 위해서 src/typing.d.ts 파일을 만들어준다.

`src/typings.d.ts`

```typescript
declare module '*.mdx';
```

## Storybook 설정

Storybook 은 5.2 버전까지 와 5.3 이후와 설정이 다르다.

현재 기준은 5.3 이후로 작성하였다.

storybook 설정은 앱 루트 폴더에 만들어지는 `.storybook` 폴더에 설정파일이 존재한다.

우선 `main.js` 를 살펴보자.

``` javascript
module.exports = {
  "stories": [
    "../src/**/*.stories.mdx",
    "../src/**/*.stories.@(js|jsx|ts|tsx)"
  ],
  "addons": [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/preset-create-react-app"
  ]
}
```

현재 module 에 존재하는 `stories` 는 story 파일 경로를 가지고 있습니다. 경로는 `glob` 모듈의 패턴에 따라 사용하면 된다.

addons는 storybook 확장 기능을 임포트 해주는 곳이다.

> addon 기능 중 addon-docs의 경우 항상 addon-essentials 보다 상위에 선언해주면 스토리북 실행시 warning 을 안보게 될 것이다.

## Story 파일 작성하기

5.3 version 부터는 `storiesOf` API 대신 [Component Story Format(CSF)](https://storybook.js.org/docs/formats/component-story-format/) 방식을 권장한다.

### Story 작성 예제

1. 컴포넌트 파일 작성

```react
//./src/stories/Button.tsx
import React from 'react';
import './button.css';

export interface ButtonProps {
  /**
   * Is this the principal call to action on the page?
   */
  primary?: boolean;
  /**
   * What background color to use
   */
  backgroundColor?: string;
  /**
   * How large should the button be?
   */
  size?: 'small' | 'medium' | 'large';
  /**
   * Button contents
   */
  label: string;
  /**
   * Optional click handler
   */
  onClick?: () => void;
}

/**
 * Primary UI component for user interaction
 */
export const Button: React.FC<ButtonProps> = ({
  primary = false,
  size = 'medium',
  backgroundColor,
  label,
  ...props
}) => {
  const mode = primary ? 'storybook-button--primary' : 'storybook-button--secondary';
  return (
    <button
      type="button"
      className={['storybook-button', `storybook-button--${size}`, mode].join(' ')}
      style={{ backgroundColor }}
      {...props}
    >
      {label}
    </button>
  );
};

```

2. 스토리 파일 작성

```react
import React from 'react';
// also exported from '@storybook/react' if you can deal with breaking changes in 6.1
import { Story, Meta } from '@storybook/react/types-6-0';

import { Button, ButtonProps } from './Button';

export default {
  title: 'Example/Button',
  component: Button,
  argTypes: {
    backgroundColor: { control: 'color' },
  },
} as Meta;

const Template: Story<ButtonProps> = (args) => <Button {...args} />;

export const Primary = Template.bind({});
Primary.args = {
  primary: true,
  label: 'Button',
};

export const Secondary = Template.bind({});
Secondary.args = {
  label: 'Button',
};

export const Large = Template.bind({});
Large.args = {
  size: 'large',
  label: 'Button',
};

export const Small = Template.bind({});
Small.args = {
  size: 'small',
  label: 'Button',
};

```

## Storybook 에 Add-on 적용하기

기본적으로 Storybook 에서 Add-on 은 `.storybook` 폴더에 설정파일인 `main.js` 에 경로를 추가해주어야 한다.

```javascript
module.exports = {
	...
  "addons": [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/preset-create-react-app"
    "@storybook/addon-knobs" // knobs 추가
  ]
}
```

참고자료

> https://velog.io/@velopert/design-system-using-typescript-and-storybook
>
> https://velog.io/@velopert/start-storybook
