# Generating tests with IA 

## unit testing

Given these guidelines: 

The Test anatomy

We include 3 parts in each test:
- What is being tested? For example, the ProductsService.addNewProduct method
- Under what circumstances and scenario? For example, no price is passed to the method
- What is the expected result? For example, the new product is not approved
```
describe('Products Service', function() { //1. unit under test
  describe('Add new product', function() { // 2. scenario
    // 3. expectation
    it('When no price is specified, then the product status is pending approval', ()=> {
      const newProduct = new ProductService().add(...);
      expect(newProduct.status).to.equal('pendingApproval');
    });
  });
});
```

Structure tests by the AAA pattern

Structure your tests with 3 well-separated sections Arrange, Act & Assert (AAA). Following this structure guarantees that the reader spends no brain-CPU on understanding the test plan:
1st A - Arrange: All the setup code to bring the system to the scenario the test aims to simulate. This might include instantiating the unit under the test constructor, adding DB records, mocking/stubbing on objects, and any other preparation code
2nd A - Act: Execute the unit under test. Usually 1 line of code
3rd A - Assert: Ensure that the received value satisfies the expectation. Usually 1 line of code

```
describe("Customer classifier", () => {
  test("When customer spent more than 500$, should be classified as premium", () => {
    //Arrange
    const customerToClassify = { spent: 505, joined: new Date(), id: 1 };
    const DBStub = sinon.stub(dataAccess, "getCustomer").reply({ id: 1, classification: "regular" });
    //Act
    const receivedClassification = customerClassifier.classifyCustomer(customerToClassify);
    //Assert
    expect(receivedClassification).toMatch("premium");
  });
});
```

And given this function to test: 
```
 async getUserUsedStorageIncrementally(user: User) {
    const mostRecentUsage = await this.usageService.getUserMostRecentUsage(
      user.uuid,
    );
    const calculateFirstDelta = !mostRecentUsage;

    if (calculateFirstDelta) {
      await this.usageService.createFirstUsageCalculation(user.uuid);
      // TODO: uncomment this when incremental usage calculation is ready
      //return this.fileRepository.sumExistentFileSizes(user.id);
      return;
    }

    const nextDeltaStartDate = mostRecentUsage.getNextPeriodStartDate();
    const hasYesterdaysUsage = Time.isToday(nextDeltaStartDate);
    if (!hasYesterdaysUsage) {
      await this.backfillUsageUntilYesterday(user, nextDeltaStartDate);
    }

    // TODO: add calculation of the current day and sum of all the usages
  }
```
Code the rest of the test cases using Jest & Sinon. Try to provide a 100% branch coverage. 

