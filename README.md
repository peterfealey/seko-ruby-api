# Seko

[![Build Status](https://travis-ci.org/jGRUBBS/seko-ruby-api.svg?branch=master)](https://travis-ci.org/jGRUBBS/seko-ruby-api.svg?branch=master)
[![Code Climate](https://codeclimate.com/github/jGRUBBS/seko-ruby-api/badges/gpa.svg)](https://codeclimate.com/github/jGRUBBS/seko-ruby-api)
[![Test Coverage](https://codeclimate.com/github/jGRUBBS/seko-ruby-api/badges/coverage.svg)](https://codeclimate.com/github/jGRUBBS/seko-ruby-api)

Ruby wrapper for Seko Logistics' SupplyStream iHub REST API v1

[SupplyStream REST API Documentation](https://wiki.supplystream.com/GetFile.aspx?Page=MANUAL.Integration-Hub-Rest-APIs&File=integration-ihub-rest-apis-v1.4.pdf)

## Integrations

1.  **Inbound Product Master Upload and method**
2.  **Inbound Companies Upload and method**
3.  **Inbound Advanced Shipment Notification**
4.  **Inbound Sales Order / Cancel Orders**
5.  **Retrieve GRN’s**
6.  **Retrieve Stock Quantity**
7.  **Retrieve Tracking Details**
8.  **Retrieve Sales Order Status**
9.  **Retrieve Stock Adjustments**
10. **Retrieve Stock Movements**

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'seko'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install seko

## Usage
#### Configuration
config/initializers/seko.rb
```ruby
Seko.configure(
  token:                'SekoAPIToKeN'
  supplier_code:        'DEFSUPLJLTD001',
  supplier_description: 'Default Supplier LARSSON & JENNINGS LTD',
  supplier_uom:         1,
  warehouses: {
    us: 'US123',
    uk: 'UK123'
  }
)
```

#### Submit Product

```ruby
client   = Seko::Client.new(Seko.config[:token])
response = client.submit_product(upc: "123456", description: 'A test product')
```

#### Submit Receipt

```ruby
line_items = [ { upc: "123456", quantity: 10 } ]
warehouse  = Seko.config[:warehouses][:us]
client     = Seko::Client.new(Seko.config[:token])
response   = client.submit_receipt(line_item_array, warehouse)
```

#### Submit Company

```ruby
company_hash = {
  code:        'IND001',
  description: 'Indigina'
}
client     = Seko::Client.new(Seko.config[:token])
response   = client.submit_company(company_hash)
```

#### Get Stock

```ruby
client   = Seko::Client.new(Seko.config[:token])
response = client.get_inventory
```

#### Check GRN

```ruby
client   = Seko::Client.new(Seko.config[:token])
response = client.check_grn('5b2dcd8e-52c3-4e27-a712-eaacda2dd8fe')
```

#### Order Status

```ruby
client   = Seko::Client.new(Seko.config[:token])
response = client.order_status('5b2dcd8e-52c3-4e27-a712-eaacda2dd8fe')
```

#### Order Tracking

```ruby
client   = Seko::Client.new(Seko.config[:token])
response = client.order_tracking('5b2dcd8e-52c3-4e27-a712-eaacda2dd8fe')
```

#### Cancel Order

```ruby
Seko::Order::CANCEL_CODES
# => {
#      "001" => "Customer Request", 
#      "002" => "Order Delayed", 
#      "003" => "Duplicate", 
#      "004" => "Item not available", 
#      "005" => "Cannot ship to address", 
#      "006" => "Other"
#    }

client   = Seko::Client.new(Seko.config[:token])
response = client.cancel_order('5b2dcd8e-52c3-4e27-a712-eaacda2dd8fe', '001')
```

#### Stock Movements

```ruby
client    = Seko::Client.new(Seko.config[:token])
warehouse = Seko.config[:warehouses][:us]
from      = 3.days.ago
to        = Time.now
response  = client.stock_movements(from, to, warehouse)
```

#### Stock Adjustments

```ruby
client    = Seko::Client.new(Seko.config[:token])
warehouse = Seko.config[:warehouses][:us]
from      = 3.days.ago
to        = Time.now
response  = client.stock_adjustments(from, to, warehouse)
```

#### Dispatch Statuses

```ruby
client    = Seko::Client.new(Seko.config[:token])
warehouse = Seko.config[:warehouses][:us] # warehouse is optional
from      = 3.days.ago
to        = Time.now
response  = client.dispatch_statuses(from, to, warehouse)

# get collection of GUIDs dispatched
guid_array = Seko::Dispatch.parse(response)
# returns something like this
# => ["2b5e52cc-fb6f-4ea4-b8cf-cf64e3a2b8db", "93e92ca2-725a-46f8-90dd-43a16105f78d"]
```

#### Submit Order

```ruby
order = {
  shipping_address: {
    first_name: "John",
    last_name:  "Smith",
    address1:   "123 Here Now",
    address2:   "2nd Floor",
    city:       "New York",
    state:      "New York",
    country:    "US",
    zipcode:    "10012",
    phone:      "123-123-1234"
  },
  email:        "someone@somehwere.com",
  number:       "R123123123",
  warehouse:    "DC123",
  date:         "2013-12-12",
  line_items: [
    {
      quantity: "1",
      sku:      "123332211"
    }
  ]
}

client   = Seko::Client.new(Seko.config[:token])
response = client.send_order_request(order)

### NOTE you may want to store the GUID from the response
### you need the guid for status updates, tracking, and canceling orders
response.guid

if response.success?
  # DO SOMETHING
else
  # QUEUE REQUEST, STORE AND RAISE ERRORS
end
```

## Contributing

1. Fork it ( https://github.com/[my-github-username]/seko/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
