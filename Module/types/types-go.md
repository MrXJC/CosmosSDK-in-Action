# types.go
> Loc: x/stake/types.go

```go
// CandidateStatus - status of a validator-candidate
type CandidateStatus byte

const (
	// nolint
	Bonded   CandidateStatus = 0x00
	Unbonded CandidateStatus = 0x01
	Revoked  CandidateStatus = 0x02
)

// Candidate defines the total amount of bond shares and their exchange rate to
// coins. Accumulation of interest is modelled as an in increase in the
// exchange rate, and slashing as a decrease.  When coins are delegated to this
// candidate, the candidate is credited with a DelegatorBond whose number of
// bond shares is based on the amount of coins delegated divided by the current
// exchange rate. Voting power can be calculated as total bonds multiplied by
// exchange rate.
type Candidate struct {
	Status               CandidateStatus `json:"status"`                 // Bonded status
	Address              sdk.Address     `json:"owner"`                  // Sender of BondTx - UnbondTx returns here
	PubKey               crypto.PubKey   `json:"pub_key"`                // Pubkey of candidate
	Assets               sdk.Rat         `json:"assets"`                 // total shares of a global hold pools
	Liabilities          sdk.Rat         `json:"liabilities"`            // total shares issued to a candidate's delegators
	Description          Description     `json:"description"`            // Description terms for the candidate
	ValidatorBondHeight  int64           `json:"validator_bond_height"`  // Earliest height as a bonded validator
	ValidatorBondCounter int16           `json:"validator_bond_counter"` // Block-local tx index of validator change
}

// Candidates - list of Candidates
type Candidates []Candidate

// NewCandidate - initialize a new candidate
func NewCandidate(address sdk.Address, pubKey crypto.PubKey, description Description) Candidate {
...
}

// Description - description fields for a candidate
type Description struct {
	Moniker  string `json:"moniker"`
	Identity string `json:"identity"`
	Website  string `json:"website"`
	Details  string `json:"details"`
}

func NewDescription(moniker, identity, website, details string) Description {
...
}

// get the exchange rate of global pool shares over delegator shares
func (c Candidate) delegatorShareExRate() sdk.Rat {
	if c.Liabilities.IsZero() {
		return sdk.OneRat()
	}
	return c.Assets.Quo(c.Liabilities)
}
```

`types.go` 就是定义模块所需的数据结构，比如上面就是stake的代码片段，定义了stake模块的`candidate`的数据结构。他的成员变量的作用我就不讲解了，和本文目前也没有很大的关系。