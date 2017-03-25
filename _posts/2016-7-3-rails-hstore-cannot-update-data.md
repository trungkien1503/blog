---
layout: post
title: It's strange - Why I can not update data with hstore on Rails
---
Yesterday, my friend asks me about his problem. He said that he updated data, other fields worked correctly but one field was not updated. He could not understand why it happened.

Luckily, I know this case, my field's type is hstore. When I change the value for a key, then I call save. It does not work. First, here is my record.

```
    #<OrderSubItem id: 178801, qty: 35, production_data: {"size"=>"008", "barcode"=>"4183013003", "dev_size"=>"M", "barcode_type"=>"128 C"}, order_item_id: 35966, sub_item_id: "NPDVR-HT001.001", created_at: "2016-06-03 07:59:51", updated_at: "2016-06-03 07:59:51", minimum_ordering_qty: 21, range: 60000, perc_extra_qty: 0.0, initial_qty: 35, sub_item_site_id: nil, production_nb: nil, pipeline_row_id: 594411, prod_data_sync_flag: false>
```

The field which I want to talk about is production_data. Its type is hstore.

```
    [4] pry(main)> osi.production_data
    => {"size"=>"008",
     "barcode"=>"4183013003",
     "dev_size"=>"M",
     "barcode_type"=>"128 C"}
```

Now our job is updating "size" from "008" to "001". It's very simple, right?

```
    [16] pry(main)> osi.production_data["size"] = "001"
    => "001"
    [17] pry(main)> osi.save
       (0.4ms)  BEGIN
      OrderItem Load (0.5ms)  SELECT  "order_items".* FROM "order_items"  WHERE "order_items"."id" = $1 LIMIT 1  [["id", 35966]]
      SubItem Load (0.4ms)  SELECT  "sub_items".* FROM "sub_items"  WHERE "sub_items"."subitem_code" = $1 LIMIT 1  [["subitem_code", "NPDVR-HT001.001"]]
       (0.3ms)  COMMIT
    => true
```

That's all. Is this correct? No, I promise. Now we could check:

```
    [18] pry(main)> osi.production_data
    => {"size"=>"001",
     "barcode"=>"4183013003",
     "dev_size"=>"M",
     "barcode_type"=>"128 C"}
```

And reload our record:

```
    [19] pry(main)> osi.reload.production_data
      OrderSubItem Load (0.6ms)  SELECT  "order_sub_items".* FROM "order_sub_items"  WHERE "order_sub_items"."id" = $1 LIMIT 1  [["id", 178801]]
    => {"size"=>"008",
     "barcode"=>"4183013003",
     "dev_size"=>"M",
     "barcode_type"=>"128 C"}
```

What's HEO? Because Rails could not see that production_data is changed.

What could we do here? We want to update data, Right? Calling attr_name_will_change! before each change to the tracked attribute is my solution for now.

```
    [21] pry(main)> osi.production_data_will_change!
    => {"size"=>"008",
     "barcode"=>"4183013003",
     "dev_size"=>"M",
     "barcode_type"=>"128 C"}
    [22] pry(main)> osi.production_data["size"] = "001"
    => "001"
    [23] pry(main)> osi.save
       (0.3ms)  BEGIN
      OrderItem Load (0.5ms)  SELECT  "order_items".* FROM "order_items"  WHERE "order_items"."id" = $1 LIMIT 1  [["id", 35966]]
      SubItem Load (0.5ms)  SELECT  "sub_items".* FROM "sub_items"  WHERE "sub_items"."subitem_code" = $1 LIMIT 1  [["subitem_code", "NPDVR-HT001.001"]]
      SQL (2.2ms)  UPDATE "order_sub_items" SET "production_data" = $1, "updated_at" = $2 WHERE "order_sub_items"."id" = 178801  [["production_data", "\"size\"=>\"001\",\"barcode\"=>\"4183013003\",\"dev_size\"=>\"M\",\"barcode_type\"=>\"128 C\""], ["updated_at", "2016-06-03 15:23:30.725504"]]
       (1.1ms)  COMMIT
    => true
```

Please check our result with reloading the record:

```
    [24] pry(main)> osi.reload.production_data
      OrderSubItem Load (0.6ms)  SELECT  "order_sub_items".* FROM "order_sub_items"  WHERE "order_sub_items"."id" = $1 LIMIT 1  [["id", 178801]]
    => {"size"=>"001",
     "barcode"=>"4183013003",
     "dev_size"=>"M",
     "barcode_type"=>"128 C"}
```

Yeah, it works. Hope it could help you on your case.

You could check more information at this link : http://api.rubyonrails.org/classes/ActiveModel/Dirty.html

*   Call define_attribute_methods passing each method you want to track.
*   Call attr_name_will_change! before each change to the tracked attribute.
*   Call changes_applied after the changes are persisted.
*   Call clear_changes_information when you want to reset the changes information.
*   Call restore_attributes when you want to restore previous data.

# Conclusion

This is a small experience when working with Rails. And this is my solution for now, you could update the data with some special cases like above case. Please push comment to give us a better solution for this case.