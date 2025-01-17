---
description: Boilerplate Encapsulated
---

# The buildState\(\) function

The `buildState` function has a fairly strait forward and simple signature that requires only two arguments: the model type of the entity you wish to build NgRx state for, and an optional object containing extra initial state:

```typescript
function buildState<
    TState extends IEntityState<TModel>, 
    TParentState, 
    TModel
>(
    type: ITModelClass<TModel>, 
    extraInitialState?: any
): IModelState<TParentState, TState, TModel> 

```

You may have noticed that there is a fairly complex generic signature here. This function returns an object that contains all the necessary bits and pieces for you to export selectors, create your own customer selectors, provide initial state to your stub reducer, and of course the facade base class.

{% hint style="info" %}
All of these types tacked onto this function ensure that each of these bits and pieces have the correct types to integrate properly with NgRx down the line. 
{% endhint %}

The first argument of the function is the model type. In this case, you actually pass in the class itself, not an instance of the class. The second argument is an object that contains any extra initial state you wish to include in addition to the initial state generated for you by the library. The signature for the model class is as follows:

```typescript
type IModelClass<TModel> = new () => TModel;
```

As indicated by this interface, all model types must be _newable_, thus enforcing the _**class**_ rather than _interface_ requirement for all entity models. Also note that the constructor signature is empty...all models must have an parameterless constructor. 

The object returned by `buildState` is defined by the `IModelState` interface. This interface encapsulates the `initialState`, `selectors` map, and `facade`, among other things:

```typescript
export interface IModelState<TParentState, TState, TModel> {
  initialState: TState;
  selectors: ISelectorMap<TParentState, TModel>;
  reducer: (state: TState) => IEntityState<TModel>;
  facade: { new (type: { new (): TModel }, store: Store<any>): IEntityFacade<TModel> };
  entityState: ((state: TParentState) => TState) | (MemoizedSelector<object, any>);
}
```

{% hint style="danger" %}
A word of warning. While the `buildState` function does return a prefabricated stub reducer for you, this was a convenience that is sadly not meant to be for any AOT build. All reducer functions must be usable within a decorator, `@NgModel,` as the reducer map into which all reducer functions go is referenced in `StoreModule.forRoot(...)` when imported into an angular module. 

The stub reducer returned by `buildState` can be useful for rapid prototyping and initial development, but once you perform a production build or any other build with AOT enabled, you will need to resort to creating your own stub reducer. 
{% endhint %}

