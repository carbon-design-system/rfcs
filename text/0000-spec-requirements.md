# Spec Requirements

## Test writing

### Tests are written as framework-agnostic
* Should support unit and integration tests
* JavaScript executed on the DOM is separate from the assertions
* Tests accept `window` and optional DOM fragment
    * DOM and window creation should be separate from the tests
    * Tests should not need to know what framework the HTML was created from
    * `window` should be used to derive `document`
    * `window` is required within the tests to perform certain functionality in the browser (e.g. input/label connection)
* individual tests or test group setup is dictated by the runner (unit setup vs integration setup)
    * allow the tests to receive a setup (e.g. before/beforeAll)
* tests understand when a set of test-able items exist and can trigger an iteration of a test for each item
    * e.g. checkboxes, radio buttons, dropdowns, accordion headers - all would need the same test ran for each similar item (same test ran for each checkbox)
    * in practice this would would mean the test would ship with a function which recognizes the elements and can loop through them, run each item through the assertions separately
    * added benefit: individual elements would create their own error(s), helping troubleshooting

### Tests are written to be portable
* Assertions do not have any dependencies outside of assertion definition (e.g. If Jest Assertions are used, the the Jest test framework does not also have to be used)
    * Both Chai or Jest assertion libraries fit this requirement
* Tests are grouped by functionality (e.g. opening an accordion (group) contains separate tests for keyboard users, mouse users, and screen reader users)
* Should allow for variable HTML structure (or attributes)
   * expands the number of tests required for a given variant
   * conditionals can be triggered by 
        * detection in the HTML
        * pre-defined configuration (nesting)
        * developer defined configuration
* Tests should be exported and nest-able
  * This would allow stacking tests together, (e.g. using the tests for text-input, textarea, and button to test a form)
  * Allows portability outside of `carbon` monorepo
    * Would allow Vue, Angular, Carbon Web Components, and PAL (handlebars) to use the component tests before being integrated into `carbon` monorepo, reducing bugs
  * accept selectors for element-level testing
    * pre-defined or developer defined
  * Allow scoping to specific tests and test groups within component (e.g. inheriting scenarios from checkbox excluding test for icons when it's used as a selectable tag)

### Test output
* Should allow, at a minimum, both `warning` and `error` 
* Output should be reflective of the requirement that was tested 
  * errors/warnings output include a message about the requirement that was being tested, not just expected/output-of-code

## Requirements writing
* Requirements document each user type's experience:
  * Generic user
  * Keyboard-only user (when user hits `ESC` xyz happens)
  * Mouse-only user (when user clicks mouse abc happens)
  * Screen-reader user (when focus is on button, screen reader should tell the user 123)
  * Touch user (this is new, but should be considered a future feature)
* Variant-level requirements
  * A variant (like a disabled button) should have separate requirements (e.g. "disable buttons can not be clicked, hovered, focused, etc")
* Legible and understandable for multiple users:
  * Designers/UX know what they're creating
  * Developers know what they're building
  * QA/IBMa know what is expected
* Single source of truth
  * Requirements should limit external references and be the primary location for any user to understand what is the expected behavior of the component or pattern
