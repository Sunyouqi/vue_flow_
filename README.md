# vue_flow_project

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build

<StcSystem id="1" serializationBase="true"
InSimulationMode="False"
UseSmbMessaging="FALSE"
ApplicationName="TestCenter"
TSharkPath=""
Active="TRUE"
LocalActive="TRUE"
Name="StcSystem 1">
<StcSystem id="1" serializationBase="true" 
 InSimulationMode="FALSE"
 UseSmbMessaging="FALSE"
 ApplicationName="TestCenter"
 TSharkPath=""
 Active="TRUE"
 LocalActive="TRUE"
 Name="StcSystem 1">
  <Project id="2" 
   TableViewData=""
   TestMode="L2L3"
   SelectedTechnologyProfiles="dhcpv6 bgp ospfv2 ospfv3 rip igmpmld"
   ConfigurationFileName="D:\ICT_study\项目测试\泰尔测试\新建文件夹\Source_spirent_file.xml"
   Active="TRUE"
   LocalActive="TRUE"
   Name="Project 1">
    <Relation type="DefaultSelection" target="5592"/>
    <Relation type="DefaultSelection" target="5626"/>
    <Tags id="2319" 
     Active="TRUE"
     LocalActive="TRUE"
     Name="Tags 1">
      <Relation type="DefaultTag" target="5586"/>
      <Relation type="DefaultTag" target="5587"/>
      <Relation type="DefaultTag" target="5588"/>
      <Relation type="DefaultTag" target="5589"/>
      <Relation type="DefaultTag" target="5590"/>
      <Relation type="DefaultTag" target="5591"/>
      <Tag id="5586" 
       Active="TRUE"
       LocalActive="TRUE"
       Name="Host">
      </Tag>
      <Tag id="5587" 
       Active="TRUE"
       LocalActive="TRUE"
       Name="Router">
      </Tag>
      <Tag id="5588" 
       Active="TRUE"
       LocalActive="TRUE"
       Name="Client">
      </Tag>
      <Tag id="5589" 
       Active="TRUE"
       LocalActive="TRUE"
       Name="Server">
      </Tag>
      <Tag id="5590" 
       Active="TRUE"
       LocalActive="TRUE"
       Name="Core">
      </Tag>
      <Tag id="5591" 
       Active="TRUE"
       LocalActive="TRUE"
       Name="Edge">
      </Tag>
    </Tags>
    <TestInfo id="2330" 
     OwnerDisplayName=""
     TestName=""
     Description=""
     UserTags=""
     WebUILaunched="FALSE"
     Active="TRUE"
     LocalActive="TRUE"
     Name="TestInfo 1">
    </TestInfo>
    <FrameLengthDistribution id="5592" 
     Seed="10900842"
     Active="TRUE"
     LocalActive="TRUE"
     Name="Default">
      <FrameLengthDistributionSlot id="5593" 
       FrameLengthMode="FIXED"
       FixedFrameLength="40"
       MinFrameLength="40"
       MaxFrameLength="41"
       Weight="7"
       Active="TRUE"
       LocalActive="TRUE"
       Name="FrameLengthDistributionSlot 5">
      </FrameLengthDistributionSlot>
      <FrameLengthDistributionSlot id="5594" 
       FrameLengthMode="FIXED"
       FixedFrameLength="576"
       MinFrameLength="575"
       MaxFrameLength="577"
       Weight="4"
       Active="TRUE"
       LocalActive="TRUE"
       Name="FrameLengthDistributionSlot 6">
      </FrameLengthDistributionSlot>
      <FrameLengthDistributionSlot id="5595" 
       FrameLengthMode="FIXED"
       FixedFrameLength="1500"
       MinFrameLength="1499"
       MaxFrameLength="1500"
       Weight="1"
       Active="TRUE"
       LocalActive="TRUE"
       Name="FrameLengthDistributionSlot 7">
      </FrameLengthDistributionSlot>
```
