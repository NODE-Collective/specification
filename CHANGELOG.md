# Changelog

We will keep a running list of changes to the spec in this document, ordered from most to least recent.

## 2025-03-05

- **Removed the `range` data type, and split fields of that type into two `number`-typed fields.**

  The range type was an attempt to reduce confusion in amounts -- specifically, to avoid the potential confusion of having `value_min` and `value_max` fields alongside `floor` and `ceiling`, as well as having `floor_min` and `floor_max`, and so on.

  The downside of that approach was poor interoperability. A "range" is not a standard data type in most tools that we expect to be used to work with NODE data, so users would have had to enter comma-separated values manually. It was also suboptimal to have a comma-separated field within a comma-separated document. Any CSV parser can handle that, but it is inelegant, and naturally suggests that the field should really be two fields.

  It became clear that that downside outweighed the upside. We reworded the descriptions of all of those fields to try to address this confusion.
