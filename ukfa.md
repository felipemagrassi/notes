# Risk oriented testing - 2023-05-15 00:06:54.361037781 -0300 -03 m=+1391.192809745

**Integration Tests** can be very slow and resource consuming and **unit tests** are not enough to ensure that the application works as expected.
The **risk-oriented #testing** is a faster way to ensure the application works as desired faster then QA can catch,
and without the full build out that integration tests need

## Example

This is a diagram that changes the method that the controller gets the information, one from a API and other from a database, how can we test this? We can use the risk-oriented testing to verify the worst cases scenarios and minimize than with testing

Asking 3 questions:

1.  Would the outcome be catastrophic if this went wrong?
2.  How likely is it that this will go wrong?
3.  If this goes wrong, is it likely to sneak through QA?

![](attachments/Pasted%20image%2020230219201956.png)

1.  When switching from the repository to service, the service data source could be possibly missing methods or returning a result that doesn't match the expected result. This is not catastrophic, but it is a problem that could be caught by unit tests.

```ruby
class ReceiptDataSourceEquivalence
  def initialize
    @repository_receipt_data_source ||= Repositories::ReceiptDataSource.new
    @service_receipt_data_source ||= Services::ReceiptDataSource.new
  end

  def local_records
    @repository_receipt_data_source.index.collect(&:id)
  end

  def api_records
    @service_receipt_data_source.index.collect(&:id)
  end

  def on_records
    Array(@local_records & @api_records)
  end
end

RSpec.describe ReceiptDataSourceEquivalence do
  before(:suite) do
      @repository_receipt_data_source ||= Repositories::ReceiptDataSource.new
      @service_receipt_data_source ||= Services::ReceiptDataSource.new

      @local_records = @repository_receipt_data_source.all.collect(&:id)
      @api_records = @service_receipt_data_source.all.collect(&:id)
      @on_records = Array(@local_records & @api_records)
  end

  describe "receipts list call" do
    it "provides receipts with the same set of ids, each of which have equivalent numbers" do
      expect(Set.new(local_records)).to eq(Set.new(api_records))
    end
  end

  describe "receipts detail call" do
    it "given an id, provides a receipt with the same attributes from local database or API" do 
      @on_records.each do |id|
        local_record = @repository_receipt_data_source.get(id)
        api_record = @service_receipt_data_source.get(id)

        expect local_record.number.to eq(api_record.number)
      end
    end
  end
end

RSpec.describe ReceiptDataSourceTransition do
  describe "get_receipts" do
    before do
      Receipt.delete_all
    end
    
    it "gets similarly structured responses from both repository and service" do
      #Subjects Under Test
      @repository = Repositories::ReceiptDataSource.new
      @service = Services::ReceiptDataSource.new
      @data_sources = [@repository, @service]
      
      #Given
      FactoryGirl.create(:receipt, number: 11.40)
      FactoryGirl.create(:receipt, number: 22.50)
      stub_request(:get, "#{ENV['RECEIPT_ENDPOINT']}/receipts.json").
      to_return(body: [
        {"uuid": "324-asd-423-fsd", "relational_id": 1, "number": 11.40},
        {"uuid": "asd-ewr-456-676", "relational_id": 2, "number":  22.50}
      ].to_json)
      
      #When
      @results = @data_sources.map {|ds| ds.get_receipts }
      
      #Then
      @results.each do |result|
        expect(result).to respond_to(:each)
        expect(result.length).to eq(2)
        expect(result.first).to respond_to(:number)
      end
    end
  end
end
```
