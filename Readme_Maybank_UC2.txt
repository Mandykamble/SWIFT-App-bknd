modified java file list outside uc2 pacakge:
1.copy all code related UC2 in pacakge(router,processor,integration ) : my2sguc2 or (for others may be ):myuc2

2.copy all UTIL code related UC2 in : com.ncs.pay.util.myuc2

3.com.ncs.pay.constants.CamelHeaderConstants.java for below chnages
   // D for debit C for credit
    public static final String HEADER_ACCOUNT_POSTING_TYPE="AccountPostingType";
    public static final String ACCOUNT_POSTING_TYPE_DEBIT="D";
    public static final String ACCOUNT_POSTING_TYPE_CREDIT="C";
4. com.ncs.pay.processor.domain.workflow.AccountPostingMYQueue
added credit account posting logic 
if(CamelHeaderConstants.ACCOUNT_POSTING_TYPE_CREDIT.equalsIgnoreCase
                    (message.getHeader(CamelHeaderConstants.HEADER_ACCOUNT_POSTING_TYPE, String.class))){
                accountPosting.setAcctCcy(payhubTxnMaster.getCreditAcctCurrencyCd());
                accountPosting.setAcctId(payhubTxnMaster.getCreditAcct());
                accountPosting.setAcctName(payhubTxnMaster.getCreditAcctNm());
            }else {
                accountPosting.setAcctCcy(payhubTxnMaster.getDebitAcctCurrencyCd());
                accountPosting.setAcctId(payhubTxnMaster.getDebitAcct());
                accountPosting.setAcctName(payhubTxnMaster.getDebtrAcctNm());
            }


5.com.ncs.pay.model.constants.WorkflowStageCodes.java
// added below constants 
public static final String FinalPostingMYUC2Queue = "FinalPostingMYUC2Queue" ;
public static final String FinalPostingUSUC2Queue = "FinalPostingUSUC2Queue" ; // 
public static final String FinalPostingUSOUTUC2Queue = "FinalPostingUSOUTUC2Queue";
public static final String FinalPostingSGUC2Queue = "FinalPostingSGUC2Queue" ;

6.message-samples folder has to be copied under resource folder 

7.merge application propertis from below file if application.yml is not overriden 
application_UC2_wf_v1.yml

7.1
payhub:
*****
  channels-setup:
		*****
	- channel-id: MB
        country-cd: MY
        product-configurations:
          - product-cd: MY_OUT_INTERB_INSTAPAY -**** this is used for UC-1****
            upstream-cutoff-time: NONE
          - product-cd: MY_OUT_INTERB_TRANSFER -**** is reqired for UC2****
            upstream-cutoff-time: NONE
			
      - channel-id: INTER_BANK_MY
        country-cd: US
        product-configurations:
          - product-cd: US_IN_INTERB_TRANSFER
            upstream-cutoff-time: NONE
          - product-cd: US_OUT_INTERB_TRANSFER
            upstream-cutoff-time: NONE
      - channel-id: INTER_BANK_MY
        country-cd: SG
        product-configurations:
            - product-cd: SG_IN_INTERB_TRANSFER
              upstream-cutoff-time: NONE


7.2
product-setup:
    products:
	  - product-cd: MY_OUT_INTERB_TRANSFER
        country-cd: MY
        allowed-branches:
          - MY_MAIN
        pre-processor-end-point: direct:my_myr2usd_out_preprocessor
        enable-pre-processor: true
        processor-endpoint: direct:my_myr2usd_out_processor
        enable-processor: true
        pre-processing-exception: test
        post-processor-endpoint: direct:my_myr2usd_out_postprocessor
      - product-cd: US_IN_INTERB_TRANSFER
        country-cd: US
        allowed-branches:
          - US_MAIN
        pre-processor-end-point: direct:us_myr2usd_in_preprocessor
        enable-pre-processor: true
        processor-endpoint: direct:us_myr2usd_in_processor
        enable-processor: true
        pre-processing-exception: test
        post-processor-endpoint: direct:us_myr2usd_in_postprocessor
      - product-cd: US_OUT_INTERB_TRANSFER
        country-cd: US
        allowed-branches:
          - US_MAIN
        pre-processor-end-point: direct:us_myr2usd_out_preprocessor
        enable-pre-processor: true
        processor-endpoint: direct:us_myr2usd_out_processor
        enable-processor: true
        pre-processing-exception: test
        post-processor-endpoint: direct:us_myr2usd_out_postprocessor
      - product-cd: SG_IN_INTERB_TRANSFER
        country-cd: SG
        allowed-branches:
          - SG_MAIN
        pre-processor-end-point: direct:sg_usd2usd_in_preprocessor
        enable-pre-processor: true
        processor-endpoint: direct:sg_usd2usd_in_processor
        enable-processor: true
        pre-processing-exception: test
        post-processor-endpoint: direct:sg_usd2usd_in_postprocessor

7.3
   workflow-queues:
      FinalPostingMYUC2Queue: direct:final-costing-uc2-my-queue
      FinalPostingUSUC2Queue: direct:final-costing-uc2-us-queue
      FinalPostingUSOUTUC2Queue: direct:final-costing-uc2-us-out-queue
      FinalPostingSGUC2Queue: direct:final-costing-uc2-sg-queue
	  
7.4
    workflow-queue-list:
      - queue-name: FinalPostingMYUC2Queue
        queue-value: direct:final-costing-uc2-my-queue
      - queue-name: FinalPostingUSUC2Queue
        queue-value: direct:final-costing-uc2-us-queue
      - queue-name: FinalPostingSGUC2Queue
        queue-value: direct:final-costing-uc2-sg-queue
      - queue-name: FinalPostingUSOUTUC2Queue
        queue-value: direct:final-costing-uc2-us-out-queue
7.5
  integration:
    http:
	target-inter-maybank-inbound-routing-endpoint: "direct:may-uc2-inter-bank-http-inbound"
7.6
myr:
  cash-pool-acc-myr: MYRCASHOB1357246880
  cash-pool-acc-php: PHPCASHOB1357246880
  cash-pool-acc-usd: USDCASHOB1357246880
  nostro-acc: US_MYR_ACT_0037
  
sgmaybank:
  cash-pool-acc-sgd: SGDSGCASHOB1357246999
  cash-pool-acc-myr: MYRSGCASHOB1357246999
  cash-pool-acc-usd: USDSGCASHOB1357246999
  cash-pool-acc-php: USDSGCASHOB1357246999
  nostro-acc: US_SGD_ACT_0037
  
us:
  vostro-sg-acc: USVSTRB1357246980
  vostro-my-acc: USVSTRB2394784739
  cash-pool-acc-usd: USDUSCASHOB1357246999
8:
com.ncs.pay.config.MyrProperties.java
@ConfigMapping(prefix = "myr")
public interface MyrProperties {
     String cashPoolAccMyr();
     String cashPoolAccPhp();
     String nostroAcc();

     String cashPoolAccUsd();
}
8.1 check for SgMayBankProperties.java prperties (prefix = "sgmaybank")
8.2 check USProperties.java (prefix = "us")  was before (prefix = "php")

9.this for HTTP properties added 
com.ncs.pay.config.HttpIntegrationProperties.java
     String targetInterMaybankInboundRoutingEndpoint();
	 
9.1 InboundMessagesResource.java
@GET
@Path("/details")
getMessageDetails() fixed null pointer 

@GET
@Path("/msgdetails")
getMsgInfoDetails
	 
10.insert DB script provided 
product_wf_query_v2.sql
tenant_table_query_v1.sql
user_table_query_v1.sql

11. UI changes for US and SG Tenant 
TenantDetails.ts





*******************Operation After deployment ******

1. api to hit from post man 
platform-http:/myussgtransferuc2v1