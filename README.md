# SQL-scripts
## As a SQL developer, some of the my scripts are shown below;

### Credit Schedule Amortization
The script below generate different forms of credit amortization ranging from; annuity, reducing balance etc
~~~
  USE [TheCoreBankingAzure] GO 
/****** Object:  StoredProcedure [dbo].[sp_Banking_TempSchedule1]    Script Date: 11/17/2024 9:52:42 PM ******/
SET 
  ANSI_NULLS ON GO 
SET 
  QUOTED_IDENTIFIER ON GO ALTER PROCEDURE [dbo].[sp_Banking_TempSchedule1] (
    @Tenor int, 
    @interestRate decimal(18, 4), 
    @principalBalance float = 0, 
    @startdate datetime, 
    @moratorium int, 
    @numOfInstalment float, 
    @CustCode varchar(100), 
    @ProductAcctNo varchar(100), 
    @pdTypeID varchar(100), 
    @PrdName varchar(100), 
    @CoyCode varchar(100), 
    @BranchCode varchar(100), 
    @dd int, 
    @ProdCode varchar(100), 
    @Name varchar(100), 
    @CoyClass varchar(100), 
    @Frequency decimal(18, 2), 
    @BulletFrq int, 
    @BulletType int, 
    @TenorMode int, 
    @WithInterest bit, 
    @Ref varchar(100), 
    @CurrentAcct varchar(100), 
    @FirstPayDate datetime, 
    @Fee decimal(18, 2), 
    @freqOfFee int, 
    @SetEqualDate bit, 
    @TerminalDate datetime, 
    @EOP bit, 
    @PrincipalFrequency decimal(18, 2), 
    @FixedPrincipal decimal(18, 2)
  ) as begin --declare  @FixedPrincipal decimal(18,2)=0.0
  declare @nxtPayDate date declare @noOfDays int declare @newdate date declare @Pnewdate date declare @principalRepyt float = 0 declare @newprincipalRepyt float = 0 declare @kounter int declare @interestAccrual float = 0 declare @TotalRepyt float = 0 declare @percentage float declare @interestCal float declare @principal float = 0 declare @ndate date declare @noOfDaysInyr int declare @instalment float declare @endBalance float = 0 declare @mth float declare @countBullet int declare @sumInterest float declare @checkCounter int declare @SumOFInterest float declare @Falseprincipal float = 0 declare @countInt int declare @mth1 int declare @ActualBalance float declare @TransDate date declare @WorkingPrincipal float = 0 declare @chkFrequency int declare @startDateDiff int declare @checkDate datetime declare @ArchivedInterest float = 0 declare @newInterestAmountd float = 0 declare @Occurence int = 0 declare @MonthlyDate date declare @effectDate datetime declare @PreInstalment int declare @IRP float declare @IntAmt float declare @Cal1 float declare @Cal2 float declare @Cal3 float declare @PMT float = 0 declare @CalInt float declare @IntAmt1 float declare @IRP1 float declare @SumInt float declare @EqualInstalment float = 0 declare @MoraPrincipal float = 0 declare @MoraInterest float = 0 declare @CummulativeInterest float = 0 declare @CummulativePrinRepy float = 0 declare @FeeCharged float declare @FeeCount float declare @paymentDate datetime declare @rPayment datetime declare @DayDiff int declare @payDay int declare @LastPayDay int declare @lastDate datetime declare @lastNewDate datetime declare @Pmth float declare @FeeKounter int = 1 declare @FirstPrinPayDate datetime declare @LastPayDate datetime if @Frequency = 12.00 
set 
  @chkFrequency = 1.00 else if @Frequency = 1.00 
set 
  @chkFrequency = 12.00 else if @Frequency = 3.00 
set 
  @chkFrequency = 4.00 else if @Frequency = 4.00 
set 
  @chkFrequency = 3.00 else if @Frequency = 6.00 
set 
  @chkFrequency = 2.00 else if @Frequency = 0.50 
set 
  @chkFrequency = 24.00 else if @Frequency = 0.25 
set 
  @chkFrequency = 52.00 else if @Frequency = 0.03 
set 
  @chkFrequency = 360.00 
set 
  @effectDate = @startdate 
set 
  @Pnewdate = @startdate 
set 
  @checkDate = @startdate 
set 
  @FirstPrinPayDate = @startdate 
set 
  @kounter = 1 
set 
  @countBullet = 1 
set 
  @checkCounter = 1 
set 
  @principal = @principalBalance 
set 
  @Instalment = @numOfInstalment - @moratorium 
set 
  @mth = @Frequency 
set 
  @Pmth = @PrincipalFrequency 
set 
  @countInt = 0 
set 
  @SumOFInterest = 0 
set 
  @ActualBalance = @principal 
set 
  @WorkingPrincipal = 0 
set 
  @TransDate = @newdate 
set 
  @CummulativeInterest = 0 
set 
  @CummulativePrinRepy = 0 
set 
  @FeeCount = 1 --if exists (select * from ts_Banking_UpdateSchedule where productAcctNo=@ProductAcctNo) and not  exists(select * from ts_Banking_AdjustSchedule where productAcctNo=@ProductAcctNo)
  --begin
  --    select @PreInstalment=count(*) from ts_Banking_UpdateSchedule where productAcctNo=@ProductAcctNo and paid=0 and PrincipalRepayment>0 
  --end
  --else
  --begin
  --  if @TenorMode=1
  --  begin
  --    set @PreInstalment=@Tenor/@Pmth
  --  end
  --  if @TenorMode=2
  --  begin
  --    set @PreInstalment=@Tenor*12/@Pmth
  --  end
  --end
  if @Pmth = 0 begin 
set 
  @Pmth = @Frequency end if @Pmth = @Frequency begin 
set 
  @PreInstalment = @numOfInstalment end else begin if @TenorMode = 1 begin 
set 
  @PreInstalment = @Tenor / @Pmth end if @TenorMode = 2 begin 
set 
  @PreInstalment = @Tenor * 12 / @Pmth end end --set @PreInstalment=@numOfInstalment
  declare @initDay int declare @oldDay int declare @PrinDate date 
set 
  @prinDate = @startdate declare @countDate int = 1 declare @rcountDate int = 0 declare @TotalPrinRepyt float = 0 declare @PDate datetime 
set 
  @rPayment = @FirstPayDate 
set 
  @FirstPrinPayDate = DATEAdd(mm, @moratorium, @FirstPayDate) --set @FirstPrinPayDate=DATEAdd(mm,@moratorium*@Frequency+@Frequency,@effectDate)
  --set @FirstPrinPayDate=@FirstPayDate
  delete [Credit].[tbl_BankingTempSchedule] 
where 
  productAcctNo = @ProductAcctNo if @dd = 1 begin while(
    @numOfInstalment >= @kounter 
    and @EOP = 0
  ) begin if (@kounter <= @moratorium) ------------------- If there is Moratorium-------------------------  
  begin ------------------------The First Pay Day----------------------------------------
set 
  @newdate = @FirstPayDate ----------------------Check last Date-------------------------------------------
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) end 
set 
  @paymentDate = @newdate 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @paymentDate) ----------------------Moratorium Without Interest-------------------
  if @WithInterest = 0 begin 
set 
  @principalRepyt = 0 
set 
  @interestAccrual = 0 
set 
  @ArchivedInterest = @ArchivedInterest + (
    dbo.fn_DailyAccruedInterest(
      @effectDate, @DayDiff, @PrincipalBalance, 
      @interestRate
    )
  ) end ----------------- Moratorium With Interest ----------------------    
  if @WithInterest = 1 begin 
set 
  @principalRepyt = 0 
set 
  @newprincipalRepyt = 0 
set 
  @principalBalance = @principal 
set 
  @interestAccrual = dbo.fn_DailyAccruedInterest(
    @effectDate, @DayDiff, @PrincipalBalance, 
    @interestRate
  ) end --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @interestAccrual end 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged 
set 
  @endBalance = @principalBalance - @newprincipalRepyt if @endBalance < 0 
set 
  @endBalance = 0 ------------------------------------------------populating the temporary table-------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @interestAccrual = 0 
set 
  @effectDate = @paymentDate 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @Pmth = @PrincipalFrequency ---------------------------The Next Pay Date-----------------------------------------    
  if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency >= 1 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end end else if (@kounter = @moratorium) begin ------------------------The First Pay Day----------------------------------------  
set 
  @newdate = @FirstPayDate --------------------------------------------Check last Date----------------------    
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) end 
set 
  @paymentDate = @newdate --------------------------------------------------------Check---------------------------------------------------------
  if @newdate = DATEADD(
    mm, @pmth * @countDate - @mth, @FirstPrinPayDate
  ) 
  and @moratorium = 0 begin 
set 
  @principalRepyt = @principal / @instalment 
set 
  @newprincipalRepyt = @principal / @instalment 
set 
  @Occurence = @Occurence + 1 
set 
  @countDate = @countDate + 1 end else begin 
set 
  @principalRepyt = 0 end --if @moratorium > 0
  --begin
  --  set @principalRepyt=@principal/@instalment
  --  set @newprincipalRepyt=@principal/@instalment        
  --end
set 
  @principalBalance = @principal 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @PaymentDate) 
set 
  @interestAccrual = @ArchivedInterest + (
    dbo.fn_DailyAccruedInterest(
      @effectDate, @DayDiff, @PrincipalBalance, 
      @interestRate
    )
  ) --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @interestAccrual end 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged 
set 
  @endBalance = @principalBalance - @newprincipalRepyt --------------------------------------------populating the temporary table-----------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @interestAccrual = 0 
set 
  @effectDate = @paymentDate 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @ArchivedInterest = 0 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @Pmth = @PrincipalFrequency if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency >= 1 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end end else begin ------------------------The First Pay Day----------------------------------------    
set 
  @newdate = @FirstPayDate --------------------------------------------Check last Date--------------------------------------------    
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) --set @newdate=@newdate
  end 
set 
  @paymentDate = @newdate if @newdate = DATEADD(
    mm, @pmth * @countDate - @mth, @FirstPrinPayDate
  ) begin if @FixedPrincipal = 0 begin 
set 
  @principalRepyt = @principal / @Instalment 
set 
  @newprincipalRepyt = @principal / @Instalment end if @FixedPrincipal <> 0 begin 
set 
  @principalRepyt = @FixedPrincipal end 
set 
  @Occurence = @Occurence + 1 
set 
  @countDate = @countDate + 1 end else begin 
set 
  @principalRepyt = 0 end --if @moratorium >= 0
  --begin
  --  set @principalRepyt=@principal/@instalment
  --set @newprincipalRepyt=@principal/@instalment
  --  set @Occurence=@Occurence +1  
  --end
set 
  @endBalance = @principalBalance - @principalRepyt if @kounter = @numOfInstalment begin 
select 
  @TotalRepyt = isnull(
    sum(PrincipalRepayment), 
    0
  ) 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  productAcctNo = @ProductAcctNo 
set 
  @principalRepyt = @principal - @TotalRepyt 
set 
  @newprincipalRepyt = @principal - @TotalRepyt --set @principalRepyt=0
  end 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @PaymentDate) 
set 
  @interestAccrual = @ArchivedInterest + (
    dbo.fn_DailyAccruedInterest(
      @effectDate, @DayDiff, @PrincipalBalance, 
      @interestRate
    )
  ) --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @interestAccrual end 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged ------------------------------------------------populating the temporary table-------------------------------------------------------------------------------    
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @interestAccrual = 0 
set 
  @principalBalance = @principalBalance - @principalRepyt 
set 
  @newdate = @paymentDate 
set 
  @effectDate = @paymentDate 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @ArchivedInterest = 0 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @Pmth = @PrincipalFrequency if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency >= 1 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end end end end else if @dd = 4 
  and @TenorMode <> 0 begin 
set 
  @kounter = 1 
set 
  @IRP =(@InterestRate * 0.01)/ @chkFrequency 
set 
  @Cal1 = 1 + @IRP 
set 
  @SumInt = 0 
set 
  @IntAmt1 = @IRP * @Principal 
set 
  @Cal2 = POWER(@cal1, @PreInstalment) 
set 
  @Cal3 = 1 -(1 / @Cal2) 
set 
  @EqualInstalment = @IntAmt1 / @cal3 
set 
  @WorkingPrincipal = @EqualInstalment ---------------------------------------------Pick the last Pay date----------------------------------------
  declare @noOfDaysBTWRepyt float declare @noOfDaysToRepayment float declare @nextPaymentDate datetime declare @numPreInstalment int 
set 
  @numPreInstalment = @numOfInstalment if exists(
    select 
      * 
    from 
      [Credit].[tbl_BankingUpdateSchedule] 
    where 
      productAcctNo = @ProductAcctNo
  ) begin 
set 
  @LastPayDate = dbo.fn_pickEffectiveDate(@ProductAcctNo) 
set 
  @nextPaymentDate = @FirstPayDate end else begin 
set 
  @LastPayDate = @startdate 
set 
  @nextPaymentDate = DATEADD(mm, @mth, @startdate) end while @numOfInstalment >= @kounter begin 
set 
  @noOfDaysBTWRepyt = DATEDIFF(
    dd, @LastPayDate, @nextPaymentDate
  ) 
set 
  @noOfDaysToRepayment = DATEDIFF(dd, @effectdate, @FirstPayDate) ---------------------Picking a Specific Date---------------------------
set 
  @newdate = @FirstPayDate --------------------------------------------Check last Date--------------------------------------------    
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) --set @newdate=@newdate
  end 
set 
  @ndate = dbo.fn_holidays(@newdate) 
set 
  @newdate = @newdate --set @noOfDays=dbo.fn_daysInMonths(YEAR(@newdate) ,MONTH(@newdate))
set 
  @noOfDays = DATEDIFF(dd, @effectdate, @FirstPayDate) if (@kounter <= @moratorium) begin if @WithInterest = 0 begin 
set 
  @IntAmt = 0 
set 
  @principalRepyt = 0 
set 
  @EqualInstalment = 0 
set 
  @ArchivedInterest = @ArchivedInterest + (
    @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal
  ) --set @MoraInterest=@MoraInterest +(@IRP * @principal)  
  --set @MoraPrincipal=@WorkingPrincipal-@MoraInterest  
  end if @WithInterest = 1 begin --while @Occurence < @mth
  --begin
set 
  @IntAmt = @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal 
set 
  @newInterestAmountd = @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal 
set 
  @EqualInstalment = @IntAmt 
set 
  @principalRepyt = 0 --end  
set 
  @Occurence = 0 end --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principal * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 
set 
  @FeeKounter = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principal * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @intAmt end 
set 
  @TotalRepyt = @principalRepyt + @IntAmt + @FeeCharged 
set 
  @endBalance = @principal - @principalRepyt if @endBalance < 0 
set 
  @endBalance = 0 
set 
  @paymentDate = @newdate ----------------------------------------------populating table----------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @noOfDays, @principal, @paymentDate, 
    @IntAmt, @IntAmt, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @IntAmt + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @principal = @principal - @principalRepyt 
set 
  @IntAmt = 0 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @Pmth = @PrincipalFrequency 
set 
  @effectDate = @paymentDate 
set 
  @LastPayDate = @paymentDate 
set 
  @PreInstalment = @PreInstalment -1 
set 
  @numPreInstalment = @numPreInstalment -1 if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency > 0.5 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end 
set 
  @nextPaymentDate = @FirstPayDate end else if (@kounter = @moratorium) begin --while @Occurence < @mth
  --begin
set 
  @EqualInstalment = @WorkingPrincipal 
set 
  @newInterestAmountd = @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal 
set 
  @IntAmt = @ArchivedInterest + (
    @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal
  ) --  set @principalRepyt=@WorkingPrincipal-@newInterestAmountd
set 
  @Occurence = @Occurence + 1 --end
  --set @Occurence=0
  if @newdate = DATEADD(
    mm, @pmth * @countDate - @mth, @FirstPrinPayDate
  ) begin --set @principalRepyt=@noOfDaysToRepayment/@noOfDaysBTWRepyt* (@WorkingPrincipal-(@IRP * @principal))
set 
  @PMT = dbo.fn_AnnuityCalculation1(
    @principal, @interestRate, @PreInstalment, 
    @chkFrequency
  ) 
set 
  @principalRepyt = @noOfDaysToRepayment / @noOfDaysBTWRepyt * (
    @PMT -(@IRP * @principal)
  ) --set @principalRepyt=@WorkingPrincipal-(@IRP * @principal)
  --set @principalRepyt=@WorkingPrincipal-@newInterestAmountd
set 
  @Occurence = @Occurence + 1 
set 
  @countDate = @countDate + 1 
set 
  @PreInstalment = @PreInstalment -1 end --else if @newdate=@Pnewdate
  --begin
  --  set @principalRepyt=@FixedPrincipal 
  --end
  else begin 
set 
  @principalRepyt = 100 end --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principal * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 
set 
  @FeeKounter = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principal * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @intAmt end --set @TotalRepyt=@EqualInstalment+@FeeCharged
set 
  @TotalRepyt = @principalRepyt + @IntAmt + @FeeCharged 
set 
  @endBalance = @principal - @principalRepyt if @endBalance < 0 
set 
  @endBalance = 0 --------------------------------------------Check last Date--------------------------------------------    
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @newdate) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) --if @payDay>@LastPayDay
  --begin
  --    set @newdate=@lastDate    
  --end
  --else
  --  set @newdate=@lastNewDate
set 
  @paymentDate = @newdate ----------------------------------------------populating table----------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @noOfDays, @principal, @paymentDate, 
    @IntAmt, @IntAmt, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @IntAmt + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @principal = @principal - @principalRepyt 
set 
  @IntAmt = 0 
set 
  @ArchivedInterest = 0 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @Pmth = @PrincipalFrequency 
set 
  @effectDate = @paymentDate 
set 
  @LastPayDate = @paymentDate 
set 
  @numPreInstalment = @numPreInstalment -1 if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency > 0.5 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end 
set 
  @nextPaymentDate = @FirstPayDate end else begin --while @Occurence < @mth
  --begin
  --end
set 
  @EqualInstalment = @WorkingPrincipal 
set 
  @newInterestAmountd = @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal 
set 
  @IntAmt = @ArchivedInterest + (
    @noOfDaysToRepayment / @noOfDaysBTWRepyt * @IRP * @principal
  ) if @newdate = DATEADD(
    mm, @pmth * @countDate - @mth, @FirstPrinPayDate
  ) begin if @FixedPrincipal = 0 begin 
set 
  @PMT = dbo.fn_AnnuityCalculation1(
    @principal, @interestRate, @PreInstalment, 
    @chkFrequency
  ) --set @principalRepyt=@noOfDaysToRepayment/@noOfDaysBTWRepyt*(@WorkingPrincipal-(@IRP * @principal))
set 
  @principalRepyt = @noOfDaysToRepayment / @noOfDaysBTWRepyt * (
    @PMT -(@IRP * @principal)
  ) --set @principalRepyt=@WorkingPrincipal-(@IRP * @principal)
  --set @principalRepyt=@WorkingPrincipal-@newInterestAmountd
  end if @FixedPrincipal <> 0 begin 
set 
  @principalRepyt = @FixedPrincipal end 
set 
  @Occurence = @Occurence + 1 
set 
  @countDate = @countDate + 1 
set 
  @PreInstalment = @PreInstalment -1 end else begin 
set 
  @principalRepyt = 0 end 
set 
  @endBalance = @principal - @principalRepyt if @kounter = @numOfInstalment begin 
select 
  @TotalRepyt = isnull(
    sum(PrincipalRepayment), 
    0
  ) 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  productAcctNo = @ProductAcctNo 
set 
  @principalRepyt = @principalBalance - @TotalRepyt 
set 
  @principalRepyt = @principalBalance - @TotalRepyt 
set 
  @endBalance = @principal - @principalRepyt --set @principalRepyt=0
  end 
set 
  @Occurence = 0 --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principal * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 
set 
  @FeeKounter = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principal * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @intAmt end 
set 
  @TotalRepyt = @principalRepyt + @IntAmt + @FeeCharged --------------------------------------------Check last Date--------------------------------------------    
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @newdate) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) --if @payDay>@LastPayDay
  --begin
  --    set @newdate=@lastDate    
  --end
  --else
  --  set @newdate=@lastNewDate
  if @endBalance < 0 
set 
  @endBalance = 0 
set 
  @paymentDate = @newdate ----------------------------------------------populating table----------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @noOfDays, @principal, @paymentDate, 
    @IntAmt, @IntAmt, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @IntAmt + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------            
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @principal = @principal - @principalRepyt 
set 
  @IntAmt = 0 
set 
  @ArchivedInterest = 0 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @effectDate = @paymentDate 
set 
  @LastPayDate = @paymentDate 
set 
  @numPreInstalment = @numPreInstalment -1 if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency > 0.5 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end 
set 
  @nextPaymentDate = @FirstPayDate end end end else if @dd = 4 
  and @TenorMode = 0 begin 
set 
  @kounter = 1 
set 
  @IRP =(@InterestRate * 0.01)/ 360.000 
set 
  @Cal1 = 1 + @IRP 
set 
  @SumInt = 0 
set 
  @IntAmt1 = @IRP * @Principal 
set 
  @Cal2 = POWER(@cal1, @numOfInstalment) 
set 
  @Cal3 = 1 -(1 / @Cal2) 
set 
  @EqualInstalment = @IntAmt1 / @cal3 
set 
  @WorkingPrincipal = @EqualInstalment while @numOfInstalment >= @kounter begin if @Frequency = 0.5 begin 
set 
  @newdate = dateadd(dd, 14, @newdate) end if @Frequency = 0.25 begin 
set 
  @newdate = dateadd(dd, 7, @newdate) end if @Frequency = 0.03 begin 
set 
  @newdate = dateadd(dd, 1, @newdate) end 
set 
  @newdate = dateadd(dd, @mth, @newdate) --set @ndate=dbo.fn_holidays(@newdate)
set 
  @newdate = @newdate 
set 
  @noOfDays = dbo.fn_daysInMonths(
    YEAR(@newdate), 
    MONTH(@newdate)
  ) if (@kounter <= @moratorium) begin if @WithInterest = 0 begin 
set 
  @IntAmt = 0 
set 
  @principalRepyt = 0 
set 
  @EqualInstalment = 0 end if @WithInterest = 1 begin while @Occurence < @mth begin 
set 
  @IntAmt = @IntAmt +(@IRP * @principal) 
set 
  @principalRepyt = 0 
set 
  @Occurence = @Occurence + 1 
set 
  @EqualInstalment = @IntAmt end 
set 
  @Occurence = 0 end if @countBullet <> @BulletFrq 
  and @BulletType <> 0 begin if @BulletType = 1 
set 
  @TotalRepyt = @IntAmt if @BulletType = 2 
set 
  @TotalRepyt = @principalRepyt end if @countBullet = @BulletFrq 
  and @BulletType <> 0 begin if @BulletType = 1 
set 
  @TotalRepyt = @principalBalance + @IntAmt else if @BulletType = 2 begin 
set 
  @sumInterest = dbo.fn_AccruedInterest(
    @startdate, @Tenor, @principalBalance, 
    @interestRate
  ) 
set 
  @TotalRepyt = @SumInterest + @principalRepyt end 
set 
  @countBullet = 0 end if @BulletType = 0 begin 
set 
  @TotalRepyt = @principalRepyt + @intAmt end 
set 
  @endBalance = @principal - @principalRepyt if @endBalance < 0 
set 
  @endBalance = 0 ----------------------------------------------populating the temporary table----------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance
  ) 
VALUES 
  (
    @kounter, @noOfDays, @principalBalance, 
    @newdate, @IntAmt, @IntAmt, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance
  ) 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @principal = @principal - @principalRepyt 
set 
  @IntAmt = 0 end else if (@kounter = @moratorium) begin while @Occurence < @mth begin 
set 
  @IntAmt = @IntAmt +(@IRP * @principal) 
set 
  @principalRepyt = @WorkingPrincipal - @IntAmt 
set 
  @Occurence = @Occurence + 1 end 
set 
  @Occurence = 0 if @countBullet <> @BulletFrq 
  and @BulletType <> 0 begin if @BulletType = 1 
set 
  @TotalRepyt = @intAmt if @BulletType = 2 
set 
  @TotalRepyt = @principalRepyt end if @countBullet = @BulletFrq 
  and @BulletType <> 0 begin if @BulletType = 1 
set 
  @TotalRepyt = @principalBalance + @IntAmt else if @BulletType = 2 begin 
set 
  @sumInterest = dbo.fn_AccruedInterest(
    @startdate, @Tenor, @principalBalance, 
    @interestRate
  ) 
set 
  @TotalRepyt = @SumInterest + @principalRepyt end 
set 
  @countBullet = 0 end if @BulletType = 0 begin 
set 
  @TotalRepyt = @principalRepyt + @intAmt end 
set 
  @endBalance = @principal - @principalRepyt if @endBalance < 0 
set 
  @endBalance = 0 ----------------------------------------------populating the temporary table----------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance
  ) 
VALUES 
  (
    @kounter, @noOfDays, @principalBalance, 
    @newdate, @IntAmt, @IntAmt, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance
  ) 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @principal = @principal - @principalRepyt 
set 
  @IntAmt = 0 end else begin while @Occurence < @mth begin 
set 
  @IntAmt = @IntAmt +(@IRP * @principal) 
set 
  @principalRepyt = @WorkingPrincipal - @IntAmt 
set 
  @Occurence = @Occurence + 1 end 
set 
  @Occurence = 0 if @countBullet <> @BulletFrq 
  and @BulletType <> 0 begin if @BulletType = 1 
set 
  @TotalRepyt = @intAmt if @BulletType = 2 
set 
  @TotalRepyt = @principalRepyt end if @countBullet = @BulletFrq 
  and @BulletType <> 0 begin if @BulletType = 1 
set 
  @TotalRepyt = @principalBalance + @IntAmt else if @BulletType = 2 begin 
set 
  @sumInterest = dbo.fn_AccruedInterest(
    @startdate, @Tenor, @principalBalance, 
    @interestRate
  ) 
set 
  @TotalRepyt = @SumInterest + @principalRepyt end 
set 
  @countBullet = 0 end if @BulletType = 0 begin 
set 
  @TotalRepyt = @principalRepyt + @intAmt end 
set 
  @endBalance = @principal - @principalRepyt if @endBalance < 0 
set 
  @endBalance = 0 ----------------------------------------------populating the temporary table----------------------------------------------------------------------------------    
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance
  ) 
VALUES 
  (
    @kounter, @noOfDays, @principalBalance, 
    @newdate, @IntAmt, @IntAmt, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance
  ) 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @principal = @principal - @principalRepyt 
set 
  @IntAmt = 0 end end end -----------------------------------------------------------------------------------Unstructured Schedule-------------------------------------------------------------
  else if @dd = 5 begin declare @MonthDiff integer = DateDiff(mm, @startdate, @TerminalDate) declare @mnth int 
set 
  @kounter = 1 declare @counter int = 1 while(@kounter <= @instalment) begin if DATEPART(mm, @FirstPayDate)= 2 
  or DATEPART(mm, @FirstPayDate)= 5 
  or DATEPART(mm, @FirstPayDate)= 10 begin --set @ndate=dbo.fn_holidays(Eomonth(@startdate))
set 
  @mnth = DATEDIFF(mm, @startdate, @FirstPayDate) 
set 
  @paymentDate = @FirstPayDate 
set 
  @startdate = @FirstPayDate 
set 
  @principalRepyt = @principal / @instalment 
set 
  @newprincipalRepyt = @principal / @instalment 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @PaymentDate) 
set 
  @interestAccrual = dbo.fn_DailyAccruedInterest(
    @effectDate, @DayDiff, @PrincipalBalance, 
    @interestRate
  ) --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount begin 
set 
  @FeeCharged = @principalBalance * @Fee * @mnth * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @interestAccrual end 
set 
  @endBalance = @principalBalance - @newprincipalRepyt if @kounter = @instalment begin 
select 
  @TotalRepyt = sum(PrincipalRepayment) 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  productAcctNo = @ProductAcctNo 
set 
  @principalRepyt = @principal - @TotalRepyt 
set 
  @principalRepyt = @principal - @TotalRepyt end 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged ------------------------------------------------populating the temporary table-------------------------------------------------------------------------------    
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @interestAccrual = 0 
set 
  @effectDate = @paymentDate 
set 
  @FeeCount = @FeeCount + 1 
set 
  @principalBalance = @principalBalance - @principalRepyt 
set 
  @kounter = @kounter + 1 end 
set 
  @counter = @counter + 1 
set 
  @FirstPayDate = DateAdd(mm, 1, @FirstPayDate) end end ------------------------------------------------------------------Fixed Total Repayment----------------------------------------------------------
  if @dd = 6 begin while(
    @principalBalance > 0 
    and @EOP = 0
  ) -----------------@numOfInstalment>=@kounter and @EOP=0
  begin if (@kounter <= @moratorium) ------------------- If there is Moratorium-------------------------  
  begin ------------------------The First Pay Day----------------------------------------
set 
  @newdate = @FirstPayDate ----------------------Check last Date-------------------------------------------
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) end 
set 
  @paymentDate = @newdate 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @paymentDate) ----------------------Moratorium Without Interest-------------------
  if @WithInterest = 0 begin 
set 
  @principalRepyt = 0 
set 
  @interestAccrual = 0 
set 
  @ArchivedInterest = @ArchivedInterest + (
    dbo.fn_DailyAccruedInterest(
      @effectDate, @DayDiff, @PrincipalBalance, 
      @interestRate
    )
  ) end ----------------- Moratorium With Interest ----------------------    
  if @WithInterest = 1 begin 
set 
  @principalRepyt = 0 
set 
  @newprincipalRepyt = 0 
set 
  @principalBalance = @principal 
set 
  @interestAccrual = dbo.fn_DailyAccruedInterest(
    @effectDate, @DayDiff, @PrincipalBalance, 
    @interestRate
  ) end --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principal end 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged 
set 
  @endBalance = @principalBalance - @FixedPrincipal if @endBalance < 0 
set 
  @endBalance = 0 ------------------------------------------------populating the temporary table-------------------------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @interestAccrual = 0 
set 
  @effectDate = @paymentDate 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @Pmth = @PrincipalFrequency 
set 
  @numOfInstalment = @principalBalance / @FixedPrincipal ---------------------------The Next Pay Date-----------------------------------------    
  if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency >= 1 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end end else if (@kounter = @moratorium) begin ------------------------The First Pay Day----------------------------------------  
set 
  @newdate = @FirstPayDate --------------------------------------------Check last Date----------------------    
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) end 
set 
  @paymentDate = @newdate 
set 
  @principalBalance = @principal 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @PaymentDate) 
set 
  @interestAccrual = @ArchivedInterest + (
    dbo.fn_DailyAccruedInterest(
      @effectDate, @DayDiff, @PrincipalBalance, 
      @interestRate
    )
  ) --------------------------------------------------------Check---------------------------------------------------------
  if @newdate = DATEADD(
    mm, @pmth * @countDate - @mth, @FirstPrinPayDate
  ) 
  and @moratorium = 0 begin 
set 
  @principalRepyt = @FixedPrincipal - @interestAccrual 
set 
  @newprincipalRepyt = @FixedPrincipal - @interestAccrual 
set 
  @Occurence = @Occurence + 1 
set 
  @countDate = @countDate + 1 end else begin 
set 
  @principalRepyt = 0 end --if @moratorium > 0
  --begin
  --  set @principalRepyt=@principal/@instalment
  --  set @newprincipalRepyt=@principal/@instalment        
  --end
  --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @interestAccrual end 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged 
set 
  @endBalance = @principalBalance - @FixedPrincipal --------------------------------------------populating the temporary table-----------------------------------------------------------------
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @interestAccrual = 0 
set 
  @effectDate = @paymentDate 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @ArchivedInterest = 0 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @Pmth = @PrincipalFrequency if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency >= 1 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end end else begin ------------------------The First Pay Day----------------------------------------    
set 
  @newdate = @FirstPayDate --------------------------------------------Check last Date--------------------------------------------    
select 
  @initDay = DATEpart(dd, @rPayment) 
select 
  @oldDay = DATEpart(dd, @newdate) 
set 
  @startDateDiff = @initDay - @oldDay 
set 
  @lastNewDate = @newdate 
select 
  @lastDate = EOMONTH(@newdate) 
select 
  @payDay = DATEPART(dd, @rPayment) 
select 
  @LastPayDay = DATEPART(dd, @lastDate) if @payDay > @LastPayDay begin 
set 
  @newdate = @lastDate end else begin 
set 
  @newdate = dateadd(dd, @startDateDiff, @lastNewDate) --set @newdate=@newdate
  end 
set 
  @paymentDate = @newdate 
set 
  @DayDiff = DATEDIFF(dd, @effectDate, @PaymentDate) 
set 
  @interestAccrual = @ArchivedInterest + (
    dbo.fn_DailyAccruedInterest(
      @effectDate, @DayDiff, @PrincipalBalance, 
      @interestRate
    )
  ) if @newdate = DATEADD(
    mm, @pmth * @countDate - @mth, @FirstPrinPayDate
  ) begin 
set 
  @principalRepyt = @FixedPrincipal - @interestAccrual 
set 
  @newprincipalRepyt = @FixedPrincipal - @interestAccrual 
set 
  @Occurence = @Occurence + 1 
set 
  @countDate = @countDate + 1 end else begin 
set 
  @principalRepyt = 0 end 
set 
  @endBalance = @principalBalance - @FixedPrincipal + @interestAccrual 
set 
  @TotalRepyt = @interestAccrual + @principalRepyt + @FeeCharged if @principalBalance <= @FixedPrincipal begin 
select 
  @TotalRepyt = sum(PrincipalRepayment) 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  productAcctNo = @ProductAcctNo 
set 
  @principalRepyt = @principal - @TotalRepyt 
set 
  @newprincipalRepyt = @principal - @TotalRepyt 
set 
  @endBalance = @principalBalance - @principalRepyt --set @principalRepyt=0
  end --------------------------------------------- Fee Charge---------------------------------------------------
  if @freqOfFee = @FeeCount 
  and @freqOfFee <> @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged 
set 
  @FeeCount = 0 end else if @freqOfFee = @mth begin 
set 
  @FeeCharged = @principalBalance * @Fee * 0.01 
set 
  @TotalRepyt = @TotalRepyt + @FeeCharged end else begin 
set 
  @FeeCharged = 0 
set 
  @TotalRepyt = @principalRepyt + @interestAccrual end ------------------------------------------------populating the temporary table-------------------------------------------------------------------------------    
  insert into [Credit].[tbl_BankingTempSchedule] (
    Month, NoOfDays, principalBalance, 
    NextPayDay, InterestAccrual, InterestPayment, 
    PrincipalRepayment, TotalRepayment, 
    CustCode, productAcctNo, EndBalance, 
    FeeCharged, CummulativeInterest, 
    CummulativePrinRepyt
  ) 
VALUES 
  (
    @kounter, @DayDiff, @principalBalance, 
    @paymentDate, @interestAccrual, 
    @interestAccrual, @principalRepyt, 
    @TotalRepyt, @CustCode, @ProductAcctNo, 
    @endBalance, @FeeCharged, @interestAccrual + @CummulativeInterest, 
    @principalRepyt + @CummulativePrinRepy
  ) --------------------------------------------------------------------------------------------------------------------------------------------------------------          
select 
  @CummulativeInterest = CummulativeInterest, 
  @CummulativePrinRepy = CummulativePrinRepyt 
from 
  [Credit].[tbl_BankingTempSchedule] 
where 
  Month = @kounter 
  and productAcctNo = @ProductAcctNo 
set 
  @principalBalance = @principalBalance - @FixedPrincipal + @interestAccrual 
set 
  @interestAccrual = 0 
set 
  @newdate = @paymentDate 
set 
  @effectDate = @paymentDate 
set 
  @FeeKounter += 1 
set 
  @FeeCount = @FeeKounter * @mth 
set 
  @ArchivedInterest = 0 
set 
  @kounter = @kounter + 1 
set 
  @countBullet = @countBullet + 1 
set 
  @Pmth = @PrincipalFrequency if @Frequency = 0.5 begin 
set 
  @FirstPayDate = dateadd(dd, 14, @paymentDate) end if @Frequency = 0.25 begin 
set 
  @FirstPayDate = dateadd(dd, 7, @paymentDate) end if @Frequency = 0.03 begin 
set 
  @FirstPayDate = dateadd(dd, 1, @paymentDate) end if @Frequency >= 1 begin 
set 
  @FirstPayDate = dateadd(mm, @mth, @paymentDate) end end end end --select * from @temp  
  end
~~~
The script below drives the end-of-day process in a banking system
~~~
  USE [TheCoreBankingAzure] GO 
/****** Object:  StoredProcedure [dbo].[spd_EndOfDay]    Script Date: 11/17/2024 9:56:56 PM ******/
SET 
  ANSI_NULLS ON GO 
SET 
  QUOTED_IDENTIFIER ON GO ALTER PROCEDURE [dbo].[spd_EndOfDay] AS BEGIN 
Set 
  NoCount On 
Set 
  XACT_ABORT ON Begin Try BEGIN TRANSACTION DECLARE @date datetime DECLARE @ydate datetime DECLARE @newdate datetime DECLARE @deals varchar(25) DECLARE @product as Decimal(18, 2) DECLARE @sumproduct as Decimal(18, 2) DECLARE @year as int declare @CheckDate datetime DECLARE @iprincipal decimal(18, 2) DECLARE @interestRate decimal(18, 2) DECLARE @effectiveDate datetime DECLARE @countTemp integer DECLARE @kounter integer DECLARE @DealID varchar(100) DECLARE @LastEOD datetime declare @Tenor integer declare @iaccruedTodate decimal(18, 2) declare @startingDate datetime declare @countDown integer declare @BackDate bit declare @NextWorkingDate datetime declare @DayDiff int declare @today datetime declare @CoyCode varchar(100) declare @Time varchar(100) declare @Ref varchar(100) declare @ErrorMsg varchar(max)= '', 
  @ErrorSp_Name varchar(500), 
  @ErrorLine int, 
  @ErrorNumber int, 
  @ErrorState int 
select 
  @coycode = COMPANYID 
from 
  [Customer].[TBL_COMPANY] DECLARE @cond bit 
SELECT 
  @cond = endofday 
FROM 
  GeneralSetup.tbl_MutuallyExclusive 
set 
  @kounter = 1 
set 
  @iaccruedTodate = 0 
set 
  @countDown = 0 
SELECT 
  @date = CurrentDate 
FROM 
  Finance.tbl_FinanceCurrentDate 
SELECT 
  @today = CurrentDate 
FROM 
  Finance.tbl_FinanceCurrentDate 
SELECT 
  @ydate = endyear, 
  @year = [year] 
FROM 
  Generalsetup.tbl_Finance_Year --SELECT @date = CurrentDate FROM ts_FinanceCurrentDate 
set 
  @NextWorkingDate = dbo.fn_holidays(
    DateAdd(dd, 1, @today)
  ) -- begin log start of operation activity
  declare @starttime time declare @endtime time declare @duration int 
select 
  @starttime = cast (
    GETDATE() as time
  ) if @cond = 0 begin ---------------------------Increaese the Maturity Date for Untreated Matured Deals----------------------
  exec spd_MMarket_UpdateDealMaturityDate 
UPDATE 
  moneymarket.tbl_MoneyDiscounts9 
SET 
  DPTDay = 0 
WHERE 
  DPTDate >= Discount 
UPDATE 
  moneymarket.tbl_MoneyInterests9 
SET 
  IPTDay = 0 
WHERE 
  iAccrualTodate >= InterestAmount 
select 
  @DayDiff = DATEDIFF(dd, @date, @NextWorkingDate) ----------------------------------------iteration-----------------------------------------------------
  while @kounter <= @DayDiff begin 
UPDATE 
  moneymarket.tbl_MoneyDiscounts9 
SET 
  CurrentDate = DATEADD(dd, @kounter, @today) 
UPDATE 
  moneymarket.tbl_MoneyInterests9 
SET 
  LastEOD = DATEADD(dd, @kounter, @today) 
set 
  @time = LTRIM(
    RIGHT(
      CONVERT(
        VARCHAR(20), 
        GETDATE(), 
        100
      ), 
      7
    )
  ) 
set 
  @Ref = dbo.fn_TransRef() -------------------------------------------------upfront Interest-----------------------------------------------
UPDATE 
  moneymarket.tbl_MoneyDiscounts9 
SET 
  Days2maturity = DATEDIFF(dd, CurrentDate, MaturityDate) 
update 
  moneymarket.tbl_MoneyDiscounts9 
set 
  UEDiscount = Discount *(
    DATEDIFF(dd, CurrentDate, MaturityDate)
  )/ Tenor 
WHERE 
  DPTDate < Discount 
  and Matured = 0 
update 
  moneymarket.tbl_MoneyDiscounts9 
set 
  EarnedDiscount = Discount - UEDiscount 
WHERE 
  DPTDate < Discount 
  and Matured = 0 
update 
  moneymarket.tbl_MoneyDiscounts9 
set 
  DPTDay = EarnedDiscount - DPTDate --set DPTDay=Discount/Tenor
WHERE 
  DPTDate < Discount 
  and Matured = 0 
update 
  moneymarket.tbl_MoneyDiscounts9 
set 
  DPTDate = DPTDate + DPTDay 
WHERE 
  DPTDate < Discount 
  and Matured = 0 -------------------------------------------------End of upfront Interest-----------------------------------------------
  -------------------------------------------------Backend Product-------------------------------------------------------
update 
  moneymarket.tbl_MoneyInterests9 
set 
  iAccrualTodate = a.Interest 
from 
  moneymarket.tbl_MoneyInterests9 m cross apply dbo.tvf_GenerateInterest(m.DealID1, @date) a 
where 
  DealID1 = m.DealID1 
  and BackDated = 1 
  and Matured = 0 
update 
  moneymarket.tbl_MoneyInterests9 
set 
  iAccrualTodate = iAccrualTodate + a.Interest 
from 
  moneymarket.tbl_MoneyInterests9 m cross apply dbo.tvf_GenerateInterest(m.DealID1, @date) a 
where 
  DealID1 = m.DealID1 
  and BackDated = 0 
  and Matured = 0 
update 
  moneymarket.tbl_MoneyInterests9 
set 
  IPTDay = iAccrualTodate - IPTDate 
where 
  Matured = 0 
update 
  moneymarket.tbl_MoneyInterests9 
set 
  IPTDate = iAccrualTodate 
where 
  Matured = 0 ---------------------------End of Backend Product--------------------------
  ---------------------Interest Capitalzation------------------------------------------------
  --exec sp_Banking_CapitaliseInterest @date,@CoyCode
  ------------------------------Daily Interest Posting Credit--------------------------------
  exec spd_Banking_EndOfDay @CoyCode, 
  @date exec spd_MMarket_DailyInterestPosting @date ------------Run Account Sweep------------------
  Exec [dbo].[spProcessAccountSweep] @date -----------------Fee Amortization---------------------------------------
  --exec sp_LoanFeeAmortization @CoyCode,@date 
  ---------------------Overdue Loans---------------------------------------
  --exec sp_Banking_InsertIntoMaturedLoans @date
  ---------------------------Daily Interest Posting Money Market-----------------------------
set 
  @date = DATEADD(dd, 1, @date) exec spBanking_CreditAnniversaryForceDebit @date, 
  @CoyCode, 
  @Ref, 
  'sys', 
  @Time exec spBanking_CreditAnniversaryForceDebitForUnscheduled @date, 
  @CoyCode, 
  '', 
  'sys', 
  @Time exec spBanking_EndofMonthInterestCapitalization @CoyCode, 
  @date exec spMMarket_TerminateMaturedDealsCTE @CoyCode, 
  @date, 
  @Time, 
  'sys', 
  1 exec spAutomateRollover @date, 
  'sys', 
  @Time --exec sp_Banking_CreditAnniversaryNOnForceDebit @date,@CoyCode
  --exec sp_Banking_ForceDebitMaturedLoans @date,@CoyCode
  --exec sp_Bond_DailyCoupontPosting @date
  --exec sp_Bond_DailyLaibilityPosting @date
  --exec sp_BondCouponPaymentLiability --@date
  --exec sp_BondCouponPayment --@date
  ------------Added to Move Standing Order Transactions------------
  Exec dbo.spMoveStandingOrder_CPT @date Exec dbo.spMoveStandingOrder_FTT @date 
set 
  @kounter += 1 end end IF DATEADD(dd, 1, @today) > @ydate BEGIN INSERT INTO Generalsetup.tbl_Finance_YearWH 
SELECT 
  TransactionDate, 
  AccountID, 
  [Description], 
  DebitAmt, 
  CreditAmt, 
  Balance, 
  AccountTypeID, 
  AccountGroupID 
From 
  vw_YearWarehousing EXEC sp_YearReset end 
UPDATE 
  GeneralSetup.tbl_MutuallyExclusive 
SET 
  startofday = 0, 
  endofday = 1 
where 
  id = 1 commit Tran 
select 
  @endtime = cast (
    GETDATE() as time
  ) 
select 
  @duration = DATEDIFF(ss, @starttime, @endtime) End Try Begin Catch If (
    XACT_STATE() = 0
  ) Commit Tran Else Rollback Tran 
Select 
  @ErrorLine = ERROR_LINE(), 
  @ErrorMsg = ERROR_MESSAGE(), 
  @ErrorNumber = ERROR_NUMBER(), 
  @ErrorSp_Name = ERROR_PROCEDURE(), 
  @ErrorState = ERROR_STATE() Insert into ts_Banking_Error_Log 
Values 
  (
    @ErrorMsg, 
    @ErrorLine, 
    @ErrorNumber, 
    @ErrorState, 
    @ErrorSp_Name, 
    @date, 
    LTRIM(
      RIGHT(
        CONVERT(
          VARCHAR(20), 
          GETDATE(), 
          100
        ), 
        7
      )
    )
  ) 
Select 
  @ErrorMsg
End Catch
end
~~~
