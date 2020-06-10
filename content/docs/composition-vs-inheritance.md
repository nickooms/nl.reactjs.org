---
id: composition-vs-inheritance
title: Compositie Versus Overerving
permalink: docs/composition-vs-inheritance.html
redirect_from:
  - "docs/multiple-components.html"
prev: lifting-state-up.html
next: thinking-in-react.html
---

React heeft een krachtig compositie model, en we bevelen aan om compositie te gebruiken in plaats van overerving om code voor meer componenten te hergebruiken.

In dit hoofdstuk zullen we een paar problemen beschouwen waarbij ontwikkelaars die beginnend zijn met React vaak grijpen naar overerving, en laten we zien hoe we ze kunnen oplossen met compositie.

## Insluiting {#containment}

Sommige componenten weten niet op voorhand wie hun kinderen worden. Dit is vooral gebruikelijk bij componenten zoals `Sidebar` of `Dialog` die generieke "boxes" vertegenwoordigen.

We raden aan dat zulke componenten de speciale `children` prop gebruiken om child elementen rechtstreeks naar hun uitvoer door te geven:

```js{4}
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

Dit staat andere componenten toe om willekeurige children door te geven door ze in de JSX te nesten:

```js{4-9}
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

**[Probeer het op CodePen](https://codepen.io/gaearon/pen/ozqNOV?editors=0010)**

Iets wat binnen de `<FancyBorder>` JSX tag staat wordt doorgegeven aan het `FancyBorder` component als een `children` prop. Aangezien `FancyBorder` `{props.children}` rendert binnen een `<div>`, zullen de doorgegeven elementen verschijnen in de uiteindelijke uitvoer.

Alhoewel het minder gebruikelijk is, kan je soms meerdere "gaten" in een component nodig hebben. In zulke gevallen mag je je eigen conventie bedenken in de plaats van `children`:

```js{5,8,18,21}
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

[**Probeer het op CodePen**](https://codepen.io/gaearon/pen/gwZOJp?editors=0010)

React elementen zoals `<Contacts />` en `<Chat />` zijn maar objecten, dus je kunt ze als props doorgeven zoals willekeurige andere gegevens. Deze aanpak kan je bekend voorkomen uit andere bibliotheken als "slots", maar in React zijn er geen beperkingen met betrekking tot wat je als props kunt doorgeven.

## Specializatie {#specialization}

Soms denken we over componenten als "speciale gevallen" van andere componenten. Bijvoorbeeld, we zouden kunnen zeggen dat een `WelcomeDialog` een speciaal geval van `Dialog` is.

In React kan dit ook worden bereikt door middel van compositie, waar een "specifieker" component een "generieker" component rendert en configureert met props:

```js{5,8,16-18}
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

[**Probeer het op CodePen**](https://codepen.io/gaearon/pen/kkEaOZ?editors=0010)

Compositie werkt even goed voor componenten gedefinieerd als classes:

```js{10,27-31}
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

[**Probeer het op CodePen**](https://codepen.io/gaearon/pen/gwZbYa?editors=0010)

## Dus Hoe Zit Het Met Overerving? {#so-what-about-inheritance}

Bij Facebook gebruiken we React in duizenden componenten en we hebben nog geen enkel geval gevonden waar we zouden aanraden om een componenten hierarchie met overerving te gebruiken.

Props en compositie geven je alle flexibiliteit die je nodig hebt om het uiterlijk en het gedrag van een component aan te passen op een expliciete en veilige manier. Onthoud dat componenten willekeurige props kunnen aannemen, waaronder primitieve waarden, React elementen of functies.

Als je niet-UI-functionaliteit wil herbruiken voor meer componenten, raden we aan om die er uit te halen naar een aparte JavaScript module. De componenten kunnen de functie, object of class importeren en gebruiken, zonder hem uit te breiden.
