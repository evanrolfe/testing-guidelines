We test using two types of tests: unit and integrations tests. All tests written should aim to have the following properties:
- thorough, i.e. covers all edge cases
- allow us to easily refactor code
- give us confidence to add new features without breaking existing functionality
- help us write new code rather than hinder us
- follows the style laid out in [http://betterspecs.org/](http://betterspecs.org/)

We do not write tests which are:
- overly specific or brittle such that they make refactoring painful
- simply a "carbon copy" of the code that already exists in the app directory i.e.

    ```ruby
    it { is_expected.to validate_presence_of :name }
    ```

  or:

    ```ruby
    it { expect(CashPurchaseItemView.table_name).to eq('vw_Stock_CashPurchaseItems_Current') }
    ```

## Unit Tests
- location: `spec/models`
- example: [spec/models/denomination_spec.rb](https://github.com/TomBellCentegra/Centega-Stock/blob/master/spec/models/denomination_spec.rb)
- implementation: [rspec model specs](https://relishapp.com/rspec/rspec-rails/docs/model-specs)

Unit tests test the responsibilities of models in isolation. They test all edge cases and business logic. Typically they will not need to touch the database but this will not always be the case. We follow these [guidelines for unit testing](https://github.com/evanrolfe/testing-guidelines/blob/master/UNIT_TESTING_GUIDE.md).

## Integration Tests
- location: `spec/requests`
- example: [spec/requests/sessions_spec.rb](https://github.com/TomBellCentegra/Centega-Stock/blob/master/spec/requests/sessions_spec.rb)
- implementation: [rspec request specs](https://relishapp.com/rspec/rspec-rails/v/3-5/docs/request-specs/request-spec)

Integration tests test the application as a whole including the database. They do not test various different scenarios in terms of business logic (thats what the unit tests are for) but simply test that every thing is "wired up" correctly.

## Factories
- location: `spec/support/factories`
- example: [spec/support/factories/denominations.rb](https://github.com/TomBellCentegra/Centega-Stock/blob/master/spec/support/factories/denominations.rb)
- implementation: [factory girl](https://github.com/thoughtbot/factory_girl)

We use factories to generate boilerplate data for use in tests. The factories should contain the [base minimum](https://robots.thoughtbot.com/factories-should-be-the-bare-minimum) data required to insert them into the database. They require association ids to be explicitly defined in order to not have an unnecessarily large object graph of associated objects created and slow down the test suite. For example in the `Denomination` factory, both `companyid` and `mediatypeid` have a default value of `nil` because they need to be explicitely defined from within the spec i.e.

```ruby
  let!(:company) { create(:company) }
  let!(:media_type) { create(:media_type, companyid: company.id) }
  let!(:denomination) do
    create(:denomination, denominationname: '', companyid: company.id, mediatypeid: media_type.id)
  end
```
