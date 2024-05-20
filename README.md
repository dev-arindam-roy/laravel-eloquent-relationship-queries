# laravel-eloquent-relationship-queries
Laravel eloquent relationship queries

### when & whereHas & orWhereHas

```php
$dataBag['vendors'] = User::with(['userRoles'])
->whereHas('userRoles', function($roleQry) {
    $roleQry->where('role_id', 5);
})
->where('status', 1)
->orderBy('first_name', 'asc')
->get();
```

```php
$data = Procurement::with([
    'procurementItems',
    'progressStatus'
])
->whereHas('procurementItems', function($query) {
    $query->where('status', 1);
})
->where('status', 1)
->orderBy('id', 'desc')
->get();
```

```php
$order = Cart::with([
    'cartItems', 
    'customer', 
    'orderStatus', 
    'procurementBtach',
    'deliveryMan',
    'deliveryStatus',
    'deliveryTimeline'
])
->whereHas('cartItems', function($query) {
    $query->where('status', 1);
})
->where('cart_number', $orderNo)
->where('status', '!=', 3)
->orderBy('id', 'desc')
->first();
```

```php
$dataBag['data'] = Purchase::with([
    'batchInfo',
    'vendorInfo',
    'purchaseProducts'
])
->when(!empty($searchText), function ($query) use ($searchText) {
    $query->whereHas('batchInfo', function ($batchQry) use ($searchText) {
        $batchQry->where('batch_no', $searchText)
            ->orWhere('name', 'LIKE', '%' . $searchText . '%')
            ->orWhere('bill_no', $searchText);
    });
    $query->orWhereHas('vendorInfo', function ($vendorQry) use ($searchText) {
        $vendorQry->where('first_name', 'like', '%' . $searchText . '%')
            ->orWhere('last_name', 'like', '%' . $searchText . '%');
    });
    return $query;
})
->where('status', '!=', 3)
->orderBy('id', 'desc')
->paginate($pagination);
```

```php
$dataBag['data'] = Batch::where('status', '!=', 3)
->when(!empty($batchSearchText), function ($query) use ($batchSearchText) {
    return $query->where('batch_no', $batchSearchText)
        ->orWhere('name', 'like', '%' . $batchSearchText . '%');
})
->orderBy('id', 'desc')
->paginate($pagination);
```

```php
$dataBag['data'] = Sale::with([
    'customerInfo',
    'saleProducts'
])
->when(!empty($searchText), function ($query) use ($searchText) {
    $query->where('invoice_no', $searchText);
    $query->orWhereHas('customerInfo', function ($batchQry) use ($searchText) {
        $batchQry->where('phone_number', $searchText)
            ->orWhere('first_name', 'LIKE', '%' . $searchText . '%')
            ->orWhere('last_name', 'LIKE', '%' . $searchText . '%')
            ->orWhere('email_id', 'LIKE', '%' . $searchText . '%');
    });
    return $query;
})
->where('status', '!=', 3)
->orderBy('id', 'desc')
->paginate($pagination);
```

### with relation select fields

```php
$batchProducts = BatchProducts::with([
    'batchInfo' => function ($selectColQry) {
        $selectColQry->select('id', 'name', 'batch_no');
    }
])
->where('product_id', $productId)
->where('status', 1)
->where('product_qty', '>', 0)
->whereHas('batchInfo', function($query) {
    $query->where('status', 1);
})
->get();
```

```php
$purchaseProduct = PurchaseProduct::with([
    'productVariantInfo' => function ($selectColQry) {
        $selectColQry->select('id', 'price', 'unit_id', 'gst_rate', 'available_stock');
    }
])
->where('batch_id', $batchId)
->where('product_id', $productId)
->where('status', 1)
->orderBy('id', 'desc')
->first();
```

### relation of relations - with of withs

```php
$data = ProcurementItems::with(['procurement' => function($itemQry) {
    $itemQry->with(['cartOrders' => function($cartQry) {
        $cartQry->with(['cartItems' => function($cartItmQry) {
            $cartItmQry->with(['productVariant']);
        }]);
    }]);
}])
->where('procurement_id', $procurementId)
->orderBy('id', 'desc')
->get();
```

```php
$dataBag['data'] = Sale::with([
    'customerInfo' => function ($userInfoQry) {
        $userInfoQry->with(['userProfile']);
    },
    'saleProducts' => function ($saleProductsQry) {
        $saleProductsQry->with([
            'unitInfo',
            'productInfo' => function ($productInfoQry) {
                $productInfoQry->with(['productBrand']);
            } 
        ]);
    }
])
->where('hash_id', $id)
->where('status', '!=', 3)
->first();
```

```php
$procurement = Procurement::with([
    'procurementItems' => function($itemQry) {
        $itemQry->where('status', 1);
        $itemQry->with([
            'progressStatus',
            'procurement',
            'associates',
            'product',
            'unit'
        ]);
    },
    'progressStatus',
    'cartOrders'
])
->where('id', $id)
->where('status', 1)
->first();
```

```php
$order = Cart::with([
    'cartItems' => function($itemQry) {
        $itemQry->with(['productVariant', 'unit']);
    }, 
    'customer', 
    'orderStatus', 
    'procurementBtach',
    'deliveryMan',
    'deliveryStatus',
    'deliveryTimeline' => function($deliQry) {
        $deliQry->with(['deliveryStatus', 'deliveryMan']);
        $deliQry->where('status', 1)->orderBy('id', 'asc');
    }
])
->whereHas('cartItems', function($query) {
    $query->where('status', 1);
})
->where('cart_number', $orderNo)
->where('status', '!=', 3)
->orderBy('id', 'desc')
->first();
```

### orWhere grouping & whereHas

```php
$dataBag['data'] = Cart::with([
    'cartItems', 
    'customer', 
    'orderStatus', 
    'procurementBtach',
    'deliveryMan',
    'deliveryStatus'
])
->where('status', '!=', 3)
->orWhere(function($query) {
    $query->whereNull('order_image');
    $query->where('order_status_id', 16);
    $query->whereHas('cartItems', function($cartQry) {
        $cartQry->where('status', 1);
    });
})
->orderBy('id', 'desc')
->paginate(25);
```
