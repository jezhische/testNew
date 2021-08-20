getInvoices(page: Page, model: InvoiceQueryModel): Observable<Pair<InvoiceLoadingStatusEnum, PagedData<InvoiceListModel>>> {
        if (!this._invoicesCache.get(this.customerId)
            || !this._invoicesCache.get(this.customerId).value.first
            || this._invoicesCache.get(this.customerId).value.first !== InvoiceLoadingStatusEnum.SUCCESS) {
            this._invoicesCache.set(this.customerId,
                new BehaviorSubject(new Pair(InvoiceLoadingStatusEnum.LOADING, new PagedData<InvoiceListModel>())));
            // this._invoicesCache.get(this.customerId).value.first = InvoiceLoadingStatusEnum.LOADING;
            return this.http.post<PagedData<InvoiceListModel>>(`${Constants.Default.API_ENDPOINT}/public-invoice/invoices/${this.getPathParam()}`, this.getRequestBody())
                .pipe(map(value => {

                    const newList = value.data.filter(value1 => {
                        switch (this.billType) {
                            case BilTypeEnum.agency_billed:
                            case BilTypeEnum.existing_bil:
                                return PublicInvoiceService.isDbInvoice(value1) === false;
                            case BilTypeEnum.direct_billed:
                                return PublicInvoiceService.isDbInvoice(value1) === true;
                            default:
                                return true;
                        }
                    });
                    const response = new ListResponse<InvoiceListModel>();
                    response.size = newList.length;
                    response.list = newList;
                    const result = new Pair(InvoiceLoadingStatusEnum.SUCCESS, getPagedData(page, response));
                    this._invoicesCache.set(this.customerId, new BehaviorSubject(result));
                    return result;

                }, error => {
                    this._invoicesCache.set(this.customerId, new BehaviorSubject(new Pair(InvoiceLoadingStatusEnum.FAILED, null)));
                    return new Pair(InvoiceLoadingStatusEnum.FAILED, null);
                }));
        } else {
            return this._invoicesCache.get(this.customerId);
        }
    }
    
get invoicesCache(): Observable<Pair<InvoiceLoadingStatusEnum, PagedData<InvoiceListModel>>> {
        return this._invoicesCache.get(this.customerId);
    }
    
private _invoicesCache = new Map<string, BehaviorSubject<Pair<InvoiceLoadingStatusEnum, PagedData<InvoiceListModel>>>>();




