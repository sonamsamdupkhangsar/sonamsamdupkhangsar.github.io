# How to get error message when a webclient results with onError handling

.onErrorResume(throwable -> {
                                LOG.error("account rest call failed: {}", throwable.getMessage());
                                if (throwable instanceof WebClientResponseException) {
                                    WebClientResponseException webClientResponseException = (WebClientResponseException) throwable;
                                    LOG.error("error body contains: {}", webClientResponseException.getResponseBodyAsString());

                                    return userRepository.deleteByAuthenticationId(userTransfer.getAuthenticationId())
                                                    .then(
                                            Mono.error(new SignupException("Account api call failed with error: " +
                                                    webClientResponseException.getResponseBodyAsString())));
                                }
                                else {
                                    return Mono.error(new SignupException("Account api call failed with error: " +throwable.getMessage()));
                                }

                            });