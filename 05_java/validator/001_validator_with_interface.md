# Validator with interface

```java
// /domain/cart/service/
public interface CartRule {
    void validate(CartItemCommand command, Member member, Cart cart);
}
```

```java
// /domain/cart/service/policy
public class ExampleA1Policy implements CartRule {

    private static final int MAX_ADVENT_CALENDAR = 1;

    @Override
    public void validate(CartItemCommand command, Member member, Cart cart) {
        var flags = command.flags();

        if (flags.isAdventCalendar()) {
            validateOnlyOne(cart, command.quantity());
            validateNoKpopMix(cart);
        }

        if (flags.isKpop()) {
            validateNoAdventCalendarMix(cart);
        }
    }
```

```java
// /domain/cart/service/CartValidator
public class CartValidator {

    private static final Logger log = LoggerFactory.getLogger(CartValidator.class);

    private final List<CartRule> rules;
    private final CartPurchaseValidator purchaseValidator;
    private PromotionVerifier promotionVerifier;

    public CartValidator(
            List<CartRule> rules,
            CartPurchaseValidator purchaseValidator) {
        this.rules = rules;
        this.purchaseValidator = purchaseValidator;
    }

    public void setPromotionVerifier(PromotionVerifier promotionVerifier) {
        this.promotionVerifier = promotionVerifier;
    }

    /**
     * Cart 상품 추가/수량 변경 시 검증 (즉시 차단)
     */
    public void validate(CartItemCommand command, Member member, Cart cart) {
        // 기존 CartRule 검증
        rules.forEach(rule -> rule.validate(command, member, cart));

        // PurchaseValidator + PromotionLimitPolicy 검증
        if (purchaseValidator != null) {
            ValidationResult result = validateForAddItem(command, member, cart);
            if (!result.valid()) {
                String errorCode = result.errors().get(0).code();
                throw new CartValidationException(errorCode, result.getFirstErrorMessage());
            }
        }
    }

    /**
     * ViewCart 검증 (에러 수집 → 보정/선별 처리)
     */
    public ValidationResult validate(
        Cart cart,
        Member member,
        CountryCode countryCode,
        Map<ProductId, Product> productMap,
        Money discountedAmountExcludingPoints
    ) {
        List<PurchaseItemDTO> itemDTOs = cart.getItems().stream()
            .filter(cartItem -> productMap.containsKey(cartItem.getProductId()))
            .map(cartItem -> {
                Product product = productMap.get(cartItem.getProductId());
                return PurchaseItemDTO.from(
                    cartItem,
                    product.getStockQuantity(),
                    product.isAvailable(),
                    product.getMinBuyQuantity(),
                    product.getMaxBuyQuantity()
                );
            }).toList();

        PurchaseValidationDTO context = PurchaseValidationDTO.forViewCart(
            cart, member, countryCode, itemDTOs, discountedAmountExcludingPoints);

        ValidationResult result = purchaseValidator.validate(context);

        if (promotionVerifier != null) {
            List<String> cartProductIds = cart.getItems().stream()
                .map(item -> item.getProductId().value()).toList();
            result = result.merge(new PromotionLimitPolicy(promotionVerifier, cartProductIds).validate(context));
        }

        return result;
    }

    private ValidationResult validateForAddItem(CartItemCommand command, Member member, Cart cart) {
        PurchaseItemDTO currentItem = PurchaseItemDTO.from(command);
        int existingQuantity = cart.getQuantityOf(command.productId());

        PurchaseValidationDTO context = PurchaseValidationDTO.forCartAddItem(
            member, currentItem, cart, existingQuantity);

        ValidationResult result = purchaseValidator.validate(context);

        if (promotionVerifier != null) {
            List<String> cartProductIds = cart.getItems().stream()
                .map(item -> item.getProductId().value()).toList();
            result = result.merge(new PromotionLimitPolicy(promotionVerifier, cartProductIds).validate(context));
        }

        return result;
    }
}
```

```java
// /application/config/DomainServiceConfig
@Configuration
public class DomainServiceConfig {

    @Bean
    public CartValidator cartValidator(ShopConfigRepository shopConfigRepository,
                                       MemberRepository memberRepository,
                                       OrderRepository orderRepository,
                                       CartPurchaseValidator cartPurchaseValidator) {
        return new CartValidator(
            Arrays.asList(
                new MemberGradePolicy(shopConfigRepository),
                new StockValidator(),
                new QuantityValidator(),
                new AdventCalendarPolicy(),
                new EventProductPolicy(orderRepository),
                new com.silicon2.stylekorean.domain.cart.service.policy.CountryRestrictionPolicy(),
                new com.silicon2.stylekorean.domain.cart.service.policy.SpecialOfferLimitPolicy(),
                new com.silicon2.stylekorean.domain.cart.service.policy.NewMemberDealLimitPolicy()
            ), cartPurchaseValidator);
    }
    ...
}
```

```java
public class CartValidator {

    private static final Logger log = LoggerFactory.getLogger(CartValidator.class);

    private final List<CartRule> rules;
    private final CartPurchaseValidator purchaseValidator;
    private PromotionVerifier promotionVerifier;

    public CartValidator(
            List<CartRule> rules,
            CartPurchaseValidator purchaseValidator) {
        this.rules = rules;
        this.purchaseValidator = purchaseValidator;
    }

    public void setPromotionVerifier(PromotionVerifier promotionVerifier) {
        this.promotionVerifier = promotionVerifier;
    }

    /**
     * Cart 상품 추가/수량 변경 시 검증 (즉시 차단)
     */
    public void validate(CartItemCommand command, Member member, Cart cart) {
        // 기존 CartRule 검증
        rules.forEach(rule -> rule.validate(command, member, cart));

        // PurchaseValidator + PromotionLimitPolicy 검증
        if (purchaseValidator != null) {
            ValidationResult result = validateForAddItem(command, member, cart);
            if (!result.valid()) {
                String errorCode = result.errors().get(0).code();
                throw new CartValidationException(errorCode, result.getFirstErrorMessage());
            }
        }
    }

    /**
     * ViewCart 검증 (에러 수집 → 보정/선별 처리)
     */
    public ValidationResult validate(
        Cart cart,
        Member member,
        CountryCode countryCode,
        Map<ProductId, Product> productMap,
        Money discountedAmountExcludingPoints
    ) {
        List<PurchaseItemDTO> itemDTOs = cart.getItems().stream()
            .filter(cartItem -> productMap.containsKey(cartItem.getProductId()))
            .map(cartItem -> {
                Product product = productMap.get(cartItem.getProductId());
                return PurchaseItemDTO.from(
                    cartItem,
                    product.getStockQuantity(),
                    product.isAvailable(),
                    product.getMinBuyQuantity(),
                    product.getMaxBuyQuantity()
                );
            }).toList();

        PurchaseValidationDTO context = PurchaseValidationDTO.forViewCart(
            cart, member, countryCode, itemDTOs, discountedAmountExcludingPoints);

        ValidationResult result = purchaseValidator.validate(context);

        if (promotionVerifier != null) {
            List<String> cartProductIds = cart.getItems().stream()
                .map(item -> item.getProductId().value()).toList();
            result = result.merge(new PromotionLimitPolicy(promotionVerifier, cartProductIds).validate(context));
        }

        return result;
    }

    private ValidationResult validateForAddItem(CartItemCommand command, Member member, Cart cart) {
        PurchaseItemDTO currentItem = PurchaseItemDTO.from(command);
        int existingQuantity = cart.getQuantityOf(command.productId());

        PurchaseValidationDTO context = PurchaseValidationDTO.forCartAddItem(
            member, currentItem, cart, existingQuantity);

        ValidationResult result = purchaseValidator.validate(context);

        if (promotionVerifier != null) {
            List<String> cartProductIds = cart.getItems().stream()
                .map(item -> item.getProductId().value()).toList();
            result = result.merge(new PromotionLimitPolicy(promotionVerifier, cartProductIds).validate(context));
        }

        return result;
    }
}
```
