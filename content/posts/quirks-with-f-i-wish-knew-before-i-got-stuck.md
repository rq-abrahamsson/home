---
title: Quirks with F# I wish knew before I got stuck
summary: e
date: 2020-04-15T19:41:11.132Z
draft: true
---


Include ThenInclude

https://github.com/dotnet/efcore/issues/6560#issuecomment-369232908





Pass functions from linq-expressions

```
    type Expr<'t> = Expressions.Expression<Func<Models.Car, 't>>
    type SortFunc<'t> = IQueryable<'t> -> Expr<obj> -> IOrderedQueryable<'t>
    type FunAs() =
        static member LinqExpression<'T, 'TResult>(e: Expression<Func<'T, 'TResult>>) = e

    let sortField (sort: Sort) (cars : IQueryable<Models.Car>) (sortFunc : SortFunc<Models.Car>) =
         match sort.field with
         | Weight -> sortFunc cars (FunAs.LinqExpression(fun s -> s.Weight :> obj))
         | Name -> sortFunc cars (FunAs.LinqExpression(fun s -> s.Name :> obj))

    let sort (sortParams : Sort) (orders : IQueryable<Models.Car>) =
         let sortFunction =
             match sortParams.order with
             | Sorting.Ascending -> fun (orders : IQueryable<Models.Car>) (f : Expressions.Expression<Func<Models.Car, obj>>)
                                        -> System.Linq.Queryable.OrderBy(orders,f)
             | Sorting.Descending -> fun (orders : IQueryable<Models.Car>) (f : Expressions.Expression<Func<Models.Car, obj>>)
                                        -> System.Linq.Queryable.OrderByDescending(orders, f)
         sortField sortParams orders sortFunction

```



F# null and Nullable

https://fsharpforfunandprofit.com/posts/the-option-type/#option-vs-null-vs-nullable