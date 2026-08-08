# Design Spec: AccountCardList

## Overview
A dataset control that renders account records as a grid of cards instead of a standard grid, for use on a subgrid.

## Component Type
Dataset Control — React Virtual Control (`ComponentFramework.ReactControl`), Fluent UI v9.

## Properties
| Name | Type (manifest `of-type`) | Usage (bound/input/output) | Required | Description |
|------|---------------------------|-----------------------------|----------|-------------|
| records | data-set | bound | true | The account records to display |
| pageSize | Whole.None | input | false | Number of cards per page, default 25 |

## Dataset Requirements (Dataset Controls only)
- Columns needed: `name` (SingleLine.Text), `revenue` (Currency)
- Pagination: required, default page size 25
- Selection/linking behavior: clicking a card calls `navigation.openForm` to open the account record

## UI Specification
- Cards laid out in a responsive Fluent UI grid, one card per account
- Card shows `name` and `revenue`
- Pagination controls at the bottom (prev/next)

## Events & Outputs
- No bound value outputs; the dataset control only triggers navigation on card click

## Test Plan
- Renders one card per record in `context.parameters.records.records`
- Next-page action calls `paging.loadNextPage()` only when `paging.hasNextPage` is true
- Clicking a card calls `navigation.openForm` with the record's entity reference
- `destroy` cleans up any listeners

## Assumptions
- Namespace: `Contoso`
- Control name: `AccountCardList`
