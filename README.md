# open-test-docc
Review the current uncommitted diff against Mukund’s comments. Do not merely reformat existing behavior.

The lifecycle-context, shutdown-timeout, conditional OTel wrapping, and SkipTLS explanation are acceptable. Two items remain:

1. Middleware-order/correlation-ID concern is not resolved.

The old enabled behavior was:

    otelhttp.NewHandler(requestLogger(slogger, h), ...)

The new code:

    logged := requestLogger(slogger, h)
    return otelhttp.NewHandler(logged, ...)

is behaviorally identical. Execution is still:

    otelhttp -> requestLogger -> cyberhttp -> panicHandler -> mux

Analyze how requestLogger creates/stores correlationId and determine the intended observability behavior.

If correlationId must be an attribute on the OTel server span, implement that explicitly using the active span rather than assuming a child request context will retroactively modify the server span. Preserve correct request logging and latency semantics. Do not blindly reorder middleware unless doing so actually satisfies the requirement.

2. Add regression coverage for both router configurations.

Add separate tests proving:
- otelEnabled=false serves requests without OTel wrapping.
- otelEnabled=true serves requests through OTel instrumentation.
- If correlationId is intended as a span attribute, install an in-memory/manual span recorder and assert the completed server span contains the expected correlation-ID attribute.

Also:
- Run gofmt on every changed Go file.
- Run the focused package tests.
- Run the full repository test command used by this project.
- Run go vet if it is part of the repository’s normal checks.
- Show the final git diff.
- Report exactly how each reviewer comment is satisfied.
- Do not commit or push.
